## [利用zookeeper实现分布式leader节点选举](https://blog.csdn.net/johnson_moon/article/details/78809995)

```
依赖原理
```

在ZK中添加基本节点，路径程序定义，节点类型为持久节点\(PERSISTENT\)。  
对需要竞选leader的每个进程，在ZK中分别添加基本节点的子节点，类型为临时自编号节点\(EPHEMERAL\_SEQUENTIAL\)，并保存创建返回的实际节点路径。  
通过delete方式删除本进程创建的子节点，可以作为退出leader状态的方式。  
基本节点的子节点类型为临时自编号节点\(EPHEMERAL\_SEQUENTIAL\)，当进程与ZK连接中断后，ZK会自动将该节点删除，确保了断连之后其他进程对leader的选举。  
由于ZK自编号产生的路径是递增的，因此可以通过判断基本节点的子节点中最小路径数字编号的节点是否是本进程新建的节点来判断是否获得leader地位。

## 原理图示 利用zk实现的分布式leader节点选举实现原理如下：

若干进程分别尝试竞选leader，情况如下：

* \(1\)8个进程分别在ZK基本节点下创建临时自编号节点，获取创建成功后的实际路径 
* \(2\)在基本节点子节点列表中，判断本进程创建节点编号是否为最小 
* \(3\)最小编号进程获得leader地位 

20171215101441232.png

leader程序异常退出或者服务器异常导致leader进程无法执行leader功能：

* \(1\)进程将ZK中对应的临时节点删除，此时基本节点下路径最小的子节点将获得leader地位

* \(2\)进程由于网络或其他原因与ZK断开了连接，ZK自动将其对应的临时节点删除

* \(3\)新出现的进程加入leader竞选，在ZK下创建临时节点，排队等待

20171215101430164.png

# 方案一 ：父节点监听方式 {#方案一-父节点监听方式}

## 实现原理 {#实现原理}

程序流程图如下：

20171215101457026.png

## 实现代码 {#实现代码}

```
package xuyihao.zktest.server.zk.leader;

import org.apache.zookeeper.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

/**
 * 基于zk的分布式leader节点选举
 * <pre>
 *     方案一：父节点监听方式
 *
 *     实现思路：监听父节点状态
 *     1.在父节点(持久化)下创建临时节点，实际创建的节点路径会根据数量进行自增(ZK自编号方式创建节点)。
 *     2.创建节点成功后，获取父节点下的子节点列表，判断本线程的路径后缀编号是否是所有子节点中最小的，若是则成为leader，反之监听父节点变动状态(通过getChildren()方法注册watcher)
 *     3.当父节点状态变动(主要是子节点列表变动)后watcher会接收到通知，这时判断父节点下的子节点的排序状态，若满足本线程的路径后缀编号最小则成为leader，反之继续注册watcher监听父节点状态
 * </pre>
 * <p>
 * Created by xuyh at 2017/11/24 9:19.
 */
public class ZKLeader {
    private static ZKLeader zkLeader;
    private Logger logger = LoggerFactory.getLogger(ZKLeader.class);
    private final static String BASE_NODE_PATH = "/ZKLeader_Leader";
    private final static String NODE_PATH = "host_process_no_";
    private String finalNodePath;

    //是否是主节点标志位
    private boolean leader = false;

    private String host = "127.0.0.1";
    private String port = "2181";
    private ZooKeeper zooKeeper;
    private FatherWatcher fatherWatcher;

    //是否连接成功标志位
    private boolean connected = false;

    public static ZKLeader create(String host, String port) {
        ZKLeader zkLeader = new ZKLeader(host, port);
        zkLeader.connectZookeeper();
        return zkLeader;
    }

    public boolean leader() {
        return leader;
    }

    public void close() {
        disconnectZooKeeper();
    }

    private ZKLeader(String host, String port) {
        this.host = host;
        this.port = port;
        this.fatherWatcher = new FatherWatcher(this);
    }

    private boolean connectZookeeper() {
        try {
            zooKeeper = new ZooKeeper(host + ":" + port, 60000, event -> {
                if (event.getState() == Watcher.Event.KeeperState.AuthFailed) {
                    leader = false;
                } else if (event.getState() == Watcher.Event.KeeperState.Disconnected) {
                    leader = false;
                } else if (event.getState() == Watcher.Event.KeeperState.Expired) {
                    leader = false;
                } else {
                    if (event.getType() == Watcher.Event.EventType.None) {//说明连接成功了
                        connected = true;
                    }
                }
            });

            int i = 1;
            while (!connected) {//等待异步连接成功,超过时间30s则退出等待
                if (i == 100)
                    break;
                Thread.sleep(300);
                i++;
            }

            if (connected) {
                if (zooKeeper.exists(BASE_NODE_PATH, false) == null) {//创建父节点
                    zooKeeper.create(BASE_NODE_PATH, "".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
                }
                //创建子节点
                finalNodePath = zooKeeper.create(BASE_NODE_PATH + "/" + NODE_PATH, "".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);

                //检查一次是否是主节点
                checkLeader();
            } else {
                logger.warn("Connect zookeeper failed. Time consumes 30 s");
                return false;
            }
        } catch (Exception e) {
            logger.warn(e.getMessage(), e);
            return false;
        }
        return true;
    }

    private boolean disconnectZooKeeper() {
        if (zooKeeper == null)
            return false;
        try {
            connected = false;
            leader = false;
            zooKeeper.close();
        } catch (Exception e) {
            logger.warn(String.format("ZK disconnect failed. [%s]", e.getMessage()), e);
        }
        return true;
    }

    private void checkLeader() {
        if (!connected)
            return;
        try {
            //获取子节点列表同时再次注册监听
            List<String> childrenList = zooKeeper.getChildren(BASE_NODE_PATH, fatherWatcher);

            if (judgePathNumMin(childrenList)) {
                leader = true;
            }
        } catch (Exception e) {
            logger.warn(e.getMessage(), e);
        }
    }

    private boolean judgePathNumMin(List<String> paths) {
        if (paths.isEmpty())
            return true;
        if (paths.size() >= 2) {
            //对无序状态的path列表按照编号升序排序
            paths.sort((str1, str2) -> {
                int num1;
                int num2;
                String string1 = str1.substring(NODE_PATH.length(), str1.length());
                String string2 = str2.substring(NODE_PATH.length(), str2.length());
                num1 = Integer.parseInt(string1);
                num2 = Integer.parseInt(string2);
                if (num1 > num2) {
                    return 1;
                } else if (num1 < num2) {
                    return -1;
                } else {
                    return 0;
                }
            });
        }

        String minId = paths.get(0);
        return finalNodePath.equals(BASE_NODE_PATH + "/" + minId);
    }


    private class FatherWatcher implements Watcher {
        private ZKLeader context;

        FatherWatcher(ZKLeader context) {
            this.context = context;
        }

        @Override
        public void process(WatchedEvent event) {
            if (event.getType() == Event.EventType.NodeChildrenChanged) {//子节点有变化
                context.checkLeader();
            }
        }
    }
}
```

### 测试程序 {#测试程序}

```
private void zkLeaderOneTestWithMultiThread() throws Exception {
    List<LeaderOneThread> leaderOneThreads = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        leaderOneThreads.add(new LeaderOneThread(ZKLeader.create("127.0.0.1", "2181"), i));
    }
    leaderOneThreads.forEach(LeaderOneThread::start);

    //线程0断连
    Thread.sleep(20000);
    leaderOneThreads.get(0).getZkLeader().close();
    Thread.sleep(2000);
    System.out.println(String.format("线程: [%s] 断开连接", 0));

    //线程1断连
    Thread.sleep(20000);
    leaderOneThreads.get(1).getZkLeader().close();
    System.out.println(String.format("线程: [%s] 断开连接", 1));

    //线程3断连
    Thread.sleep(20000);
    leaderOneThreads.get(3).getZkLeader().close();
    System.out.println(String.format("线程: [%s] 断开连接", 3));

    //线程4断连
    Thread.sleep(20000);
    leaderOneThreads.get(4).getZkLeader().close();
    System.out.println(String.format("线程: [%s] 断开连接", 4));

    //线程2断连
    Thread.sleep(20000);
    leaderOneThreads.get(2).getZkLeader().close();
    System.out.println(String.format("线程: [%s] 断开连接", 2));

    Thread.sleep(60000);
}

private class LeaderOneThread extends Thread {
    private ZKLeader zkLeader;
    private int threadNum;

    public ZKLeader getZkLeader() {
        return zkLeader;
    }

    LeaderOneThread(ZKLeader zkLeader, int threadNum) {
        this.zkLeader = zkLeader;
        this.threadNum = threadNum;
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(5000);
            } catch (Exception e) {
                e.printStackTrace();
            }
            Date dt = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String currentTime = sdf.format(dt);
            if (zkLeader.leader()) {
                System.out.println(String.format("[%s] 线程: [%s] 是主节点", currentTime, threadNum));
            }
        }
    }
}

```




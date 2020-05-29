## [利用zookeeper实现分布式leader节点选举](https://blog.csdn.net/johnson_moon/article/details/78809995)

## 依赖原理

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

![](/static/image/20171215101441232.png)

leader程序异常退出或者服务器异常导致leader进程无法执行leader功能：

* \(1\)进程将ZK中对应的临时节点删除，此时基本节点下路径最小的子节点将获得leader地位

* \(2\)进程由于网络或其他原因与ZK断开了连接，ZK自动将其对应的临时节点删除

* \(3\)新出现的进程加入leader竞选，在ZK下创建临时节点，排队等待

![](/static/image/20171215101430164.png)

# 方案一 ：父节点监听方式 {#方案一-父节点监听方式}

## 实现原理 {#实现原理}

程序流程图如下：

![](/static/image/20171215101457026.png)

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

### 结果: {#结果}

```
[2017-11-30 17:05:02] 线程: [0] 是主节点
[2017-11-30 17:05:07] 线程: [0] 是主节点
[2017-11-30 17:05:12] 线程: [0] 是主节点
线程: [0] 断开连接
[2017-11-30 17:05:22] 线程: [1] 是主节点
[2017-11-30 17:05:27] 线程: [1] 是主节点
[2017-11-30 17:05:32] 线程: [1] 是主节点
[2017-11-30 17:05:37] 线程: [1] 是主节点
线程: [1] 断开连接
[2017-11-30 17:05:42] 线程: [2] 是主节点
[2017-11-30 17:05:47] 线程: [2] 是主节点
[2017-11-30 17:05:52] 线程: [2] 是主节点
[2017-11-30 17:05:57] 线程: [2] 是主节点
线程: [3] 断开连接
[2017-11-30 17:06:02] 线程: [2] 是主节点
[2017-11-30 17:06:07] 线程: [2] 是主节点
[2017-11-30 17:06:12] 线程: [2] 是主节点
[2017-11-30 17:06:17] 线程: [2] 是主节点
线程: [4] 断开连接
[2017-11-30 17:06:22] 线程: [2] 是主节点
[2017-11-30 17:06:27] 线程: [2] 是主节点
[2017-11-30 17:06:32] 线程: [2] 是主节点
[2017-11-30 17:06:37] 线程: [2] 是主节点
线程: [2] 断开连接
[2017-11-30 17:06:42] 线程: [5] 是主节点
[2017-11-30 17:06:47] 线程: [5] 是主节点
[2017-11-30 17:06:52] 线程: [5] 是主节点
[2017-11-30 17:06:57] 线程: [5] 是主节点
[2017-11-30 17:07:02] 线程: [5] 是主节点
[2017-11-30 17:07:07] 线程: [5] 是主节点
[2017-11-30 17:07:12] 线程: [5] 是主节点
[2017-11-30 17:07:17] 线程: [5] 是主节点
[2017-11-30 17:07:22] 线程: [5] 是主节点
[2017-11-30 17:07:27] 线程: [5] 是主节点
[2017-11-30 17:07:32] 线程: [5] 是主节点
[2017-11-30 17:07:37] 线程: [5] 是主节点
```

## 方案一优劣

### 优点

实现对父节点变动状态\(主要是子节点列表变化\)的监听

当子节点列表出现变化后，ZK通知监听的各个进程，各个进程查询子节点状态

对父节点进行监听，实现起来相对简单

### 劣势

每个进程都监听父节点状态，即父节点出现变动\(主要是子节点列表变化\)后，ZK服务器需要通知到所有注册监听的进程，网络消耗和资源浪费比较大

## 方案三 ：子节点监听方式

### 实现原理

程序流程图如下：

![](/static/image/20171215101517640.png)

## 实现代码 {#实现代码-1}

```
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

/**
 * Created by xuyh at 2017/11/30 14:40.
 * <p>
 * **最优方案**
 * <pre>
 *     方案三：子节点监听方式
 *
 *     实现思路：监听子节点状态
 *     1.在父节点(持久化)下创建临时节点，实际创建的节点路径会根据数量进行自增(ZK自编号方式创建节点)。
 *     2.创建节点成功后，首先获取父节点下的子节点列表，判断本线程的路径后缀编号是否是所有子节点中最小的，若是则成为leader，反之监听本节点前一个节点(路径排序为本节点路径数字减一的节点)变动状态(通过getData()方法注册watcher)
 *     3.当监听对象状态变动(节点删除状态)后watcher会接收到通知，这时再次判断父节点下的子节点的排序状态，若满足本线程的路径后缀编号最小则成为leader，反之继续注册watcher监听前一个节点状态
 */
public class ZKLeaderTwo {
    private static ZKLeaderTwo zkLeaderTwo;
    private Logger logger = LoggerFactory.getLogger(ZKLeader.class);
    private final static String BASE_NODE_PATH = "/ZKLeader_Leader";
    private final static String NODE_PATH = "host_process_no_";
    private String finalNodePath;

    //是否是主节点标志位
    private boolean leader = false;

    private String host = "127.0.0.1";
    private String port = "2181";
    private ZooKeeper zooKeeper;
    private PreviousNodeWatcher previousNodeWatcher;

    //是否连接成功标志位
    private boolean connected = false;

    public static ZKLeaderTwo create(String host, String port) {
        ZKLeaderTwo zkLeaderTwo = new ZKLeaderTwo(host, port);
        zkLeaderTwo.connectZookeeper();
        return zkLeaderTwo;
    }

    public boolean leader() {
        return leader;
    }

    public void close() {
        disconnectZooKeeper();
    }

    private ZKLeaderTwo(String host, String port) {
        this.host = host;
        this.port = port;
        this.previousNodeWatcher = new PreviousNodeWatcher(this);
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
            zooKeeper.close();
            connected = false;
            leader = false;
        } catch (Exception e) {
            logger.warn(String.format("ZK disconnect failed. [%s]", e.getMessage()), e);
        }
        return true;
    }

    private void checkLeader() {
        if (!connected)
            return;
        try {
            //获取子节点列表，若没有成为leader，注册监听，监听对象应当是比本节点路径编号小一(或者排在前面一位)的节点
            List<String> childrenList = zooKeeper.getChildren(BASE_NODE_PATH, false);

            if (judgePathNumMin(childrenList)) {
                leader = true;//成为leader
            } else {
                watchPreviousNode(childrenList);
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

    private void watchPreviousNode(List<String> paths) {
        if (paths.isEmpty() || paths.size() == 1) {
            return;
        }
        int currentNodeIndex = paths.indexOf(finalNodePath.substring((BASE_NODE_PATH + "/").length(), finalNodePath.length()));
        String previousNodePath = BASE_NODE_PATH + "/" + paths.get(currentNodeIndex - 1);
        //通过getData方法再次注册watcher
        try {
            zooKeeper.getData(previousNodePath, previousNodeWatcher, new Stat());
        } catch (Exception e) {
            logger.warn(String.format("Previous node watcher register failed! message: [%s]", e.getMessage()), e);
        }
    }

    private class PreviousNodeWatcher implements Watcher {
        private ZKLeaderTwo context;

        PreviousNodeWatcher(ZKLeaderTwo context) {
            this.context = context;
        }

        @Override
        public void process(WatchedEvent event) {
            //节点被删除了，说明这个节点放弃了leader
            if (event.getType() == Event.EventType.NodeDeleted) {
                context.checkLeader();
            }
        }
    }
}
```

### 测试程序 {#测试程序-1}

```
private void zkLeaderTwoTestWithMultiThread() throws Exception {
    List<LeaderTwoThread> leaderTwoThreads = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        leaderTwoThreads.add(new LeaderTwoThread(ZKLeaderTwo.create("127.0.0.1", "2181"), i));
    }
    leaderTwoThreads.forEach(LeaderTwoThread::start);

    //线程0断连
    Thread.sleep(20000);
    leaderTwoThreads.get(0).getZkLeaderTwo().close();
    System.out.println(String.format("线程: [%s] 断开连接", 0));

    //线程1断连
    Thread.sleep(20000);
    leaderTwoThreads.get(1).getZkLeaderTwo().close();
    System.out.println(String.format("线程: [%s] 断开连接", 1));

    //线程3断连
    Thread.sleep(20000);
    leaderTwoThreads.get(3).getZkLeaderTwo().close();
    System.out.println(String.format("线程: [%s] 断开连接", 3));

    //线程4断连
    Thread.sleep(20000);
    leaderTwoThreads.get(4).getZkLeaderTwo().close();
    System.out.println(String.format("线程: [%s] 断开连接", 4));

    //线程2断连
    Thread.sleep(20000);
    leaderTwoThreads.get(2).getZkLeaderTwo().close();
    System.out.println(String.format("线程: [%s] 断开连接", 2));

    Thread.sleep(60000);
}

private class LeaderTwoThread extends Thread {
    private ZKLeaderTwo zkLeaderTwo;
    private int threadNum;

    public ZKLeaderTwo getZkLeaderTwo() {
        return zkLeaderTwo;
    }

    LeaderTwoThread(ZKLeaderTwo zkLeaderTwo, int threadNum) {
        this.zkLeaderTwo = zkLeaderTwo;
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
            if (zkLeaderTwo.leader()) {
                System.out.println(String.format("[%s] 线程: [%s] 是主节点", currentTime, threadNum));
            }
        }
    }
}
```

### 结果: {#结果-1}

```
[2017-11-30 16:47:41] 线程: [0] 是主节点
[2017-11-30 16:47:46] 线程: [0] 是主节点
[2017-11-30 16:47:51] 线程: [0] 是主节点
[2017-11-30 16:47:56] 线程: [0] 是主节点
线程: [0] 断开连接
[2017-11-30 16:48:01] 线程: [1] 是主节点
[2017-11-30 16:48:06] 线程: [1] 是主节点
[2017-11-30 16:48:11] 线程: [1] 是主节点
[2017-11-30 16:48:16] 线程: [1] 是主节点
线程: [1] 断开连接
[2017-11-30 16:48:21] 线程: [2] 是主节点
[2017-11-30 16:48:26] 线程: [2] 是主节点
[2017-11-30 16:48:31] 线程: [2] 是主节点
[2017-11-30 16:48:36] 线程: [2] 是主节点
线程: [3] 断开连接
[2017-11-30 16:48:41] 线程: [2] 是主节点
[2017-11-30 16:48:46] 线程: [2] 是主节点
[2017-11-30 16:48:51] 线程: [2] 是主节点
[2017-11-30 16:48:56] 线程: [2] 是主节点
线程: [4] 断开连接
[2017-11-30 16:49:01] 线程: [2] 是主节点
[2017-11-30 16:49:06] 线程: [2] 是主节点
[2017-11-30 16:49:11] 线程: [2] 是主节点
[2017-11-30 16:49:16] 线程: [2] 是主节点
线程: [2] 断开连接
[2017-11-30 16:49:21] 线程: [5] 是主节点
[2017-11-30 16:49:26] 线程: [5] 是主节点
[2017-11-30 16:49:31] 线程: [5] 是主节点
[2017-11-30 16:49:36] 线程: [5] 是主节点
[2017-11-30 16:49:41] 线程: [5] 是主节点
[2017-11-30 16:49:46] 线程: [5] 是主节点
[2017-11-30 16:49:51] 线程: [5] 是主节点
[2017-11-30 16:49:56] 线程: [5] 是主节点
[2017-11-30 16:50:01] 线程: [5] 是主节点
[2017-11-30 16:50:06] 线程: [5] 是主节点
[2017-11-30 16:50:11] 线程: [5] 是主节点
[2017-11-30 16:50:16] 线程: [5] 是主节点
```

## 方案二优劣

### 优点

实现对子节点变动状态\(排序在本进程对应节点之前的一个节点\)的监听

被监听子节点变动\(删除\)之后，ZK通知本进程执行相应操作，判断是否成为leader

相对于父节点监听方式，子节点监听方式在每一次锁释放\(或者节点变动\)时，ZK仅通知到一个进程的watcher，节省了大量的网络消耗和资源占用

### 劣势

实现方式与程序逻辑较父节点监听来说比较繁琐

## 总结比较

### 程序复杂度：

父节点监听方式 &lt; 子节点监听方式

### 网络资源消耗：

父节点监听方式 &gt;&gt; 子节点监听方式

### 程序可靠性

父节点监听方式 &lt; 子节点监听方式

### 轻量Zookeeper客户端实现

github地址：

[https://github.com/johnsonmoon/zk-client](https://github.com/johnsonmoon/zk-client)


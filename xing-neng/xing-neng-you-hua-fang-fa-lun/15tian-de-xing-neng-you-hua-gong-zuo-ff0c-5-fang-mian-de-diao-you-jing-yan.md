```
前言
```

前一段时间一直在做性能调优的工作，颇有收获。因此，简单的总结并分享下研究成果。性能调优很有趣但也是个无底洞，不可能在一篇文章全部阐述完。**这里只是提供一个方向**，以后碰到了知道可以从这些方面入手即可。具体如下：

20170528092115231.png

### **代码层面** {#代码层面}

#### **for循环中不要利用 + 号去拼接字符串** {#for循环中不要利用-号去拼接字符串}

在循环次数比较多的for循环中，我们也不要利用 + 号去拼接字符串。具体例子如下：

```
程序清单 1-1

@Test
public void test(){
    String str = "ay";
    for(int i=0;i<Integer.MAX_VALUE;i++){
         str = str + i;
    }
}
```

具体解决方法如下：

* 根据具体的业务场景，使用 StringBuffer（线程安全）或者 StringBuilder（非线程安全）
* 使用数组

```
程序清单 1-1

@Test
public void test(){
    //第一种解决方法
    StringBuilder stringBuilder = new StringBuilder(Integer.MAX_VALUE);
    //第二种解决方法
    String[] strArray = new String[Integer.MAX_VALUE + 1];
    stringBuilder.append("ay");
    strArray[0] = "ay";
    for(int i=0;i<Integer.MAX_VALUE + 1;i++){
        stringBuilder.append("al");
        strArray[i + 1] = "al";
    }
    System.out.println(stringBuilder.toString());
    System.out.println(ArrayUtils.toString(strArray));
}
```

#### **设置容量参数提高系统性能** {#设置容量参数提高系统性能}

对于 StringBuffer（线程安全）或者 StringBuilder（非线程安全），都有相应的构造方法：

```
程序清单 1-1

public StringBuilder(int capacity) {
    super(capacity);
}
```

如果我们可以事先知道需要拼接的字符串长度，设置容量参数，防止 StringBuffer 在源码内部进行一系列复杂的内存复制操作，影响性能。

如上面的

```
StringBuilder stringBuilder = new StringBuilder(Integer.MAX_VALUE);
```

#### **for循环建议写法** {#for循环建议写法}

```
for (int i = 0, int length = list.size(); i < length; i++)
```

##### **方法的返回值** {#方法的返回值}

**返回List：**

```
private List<PcsTaskDTO> sortDecisionAndBackTask(List<PcsTaskDTO> pcsTaskDTOList) throws Exception{
        if(CollectionUtils.isEmpty(pcsTaskDTOList)) return null;
}
```

解决方法：

```
private List<PcsTaskDTO> sortDecisionAndBackTask(List<PcsTaskDTO> pcsTaskDTOList) throws Exception{
        if(CollectionUtils.isEmpty(pcsTaskDTOList)) return Collections.EMPTY_LIST;
}
```

**返回Set：**

```
Collections.EMPTY_SET
```

**返回Map：**

```
Collections.EMPTY_MAP
```

**返回Boolean：**

```
Boolean.TRUE
```

#### **不要再for循环中查询数据库** {#不要再for循环中查询数据库}

**解决：**

* 根据业务场景，把for循环中的多次连接数据库查询，写到sql中去查询，既一次性查询出来
* 根据业务场景，看是否可以利用缓存，提高查询效率

#### **去掉System.out.println** {#去掉systemoutprintln}

代码部署到生产环境前，去掉全部System.out.println

#### **四种数组复制方式的性能比较和抉择** {#四种数组复制方式的性能比较和抉择}

数组copy有很多种方法，效率不一。我们先看下面具体实例：

```
程序清单 2-1    

/**
 * 测试4种数组复制效率比较
 * @date 2017/2/7.
 */
public class AyTest {

    private static final byte[] buffer = new byte[1024*10];
    static {
        for (int i = 0; i < buffer.length; i++) {
            buffer[i] = (byte) (i & 0xFF);
        }
    }
    private static long startTime;

    public static void main(String[] args) {
        startTime = System.nanoTime();
        byte[] newBuffer = new byte[buffer.length];
        for(int i=0;i<buffer.length;i++) {
            newBuffer[i] = buffer[i];
        }
        calcTime("forCopy");

        startTime = System.nanoTime();
        byte[] newBuffer2 = buffer.clone();
        calcTime("cloneCopy");

        startTime = System.nanoTime();
        byte[] newBuffer3 = Arrays.copyOf(buffer, buffer.length);
        calcTime("arraysCopyOf");

        startTime = System.nanoTime();
        byte[] newBuffer4 = new byte[buffer.length];
        System.arraycopy(buffer, 0, newBuffer, 0, buffer.length);
        calcTime("systemArraycopy");
    }

    private static void calcTime(String type) {
        long endTime = System.nanoTime();
        System.out.println(type + " cost " +(endTime-startTime)+ " nanosecond");
    }
}
```

运行结果：

```
forCopy cost 711576 nanosecond
cloneCopy cost 53490 nanosecond
arraysCopyOf cost 119946 nanosecond
systemArraycopy cost 39712 nanosecond
```

多运行几次，我们得出数组复制效率：

System.arraycopy &gt; clone &gt; Arrays.copyOf &gt; for

综上所述，当复制大量数据时，使用System.arraycopy\(\)命令。

## 8.实现高性能的字符串分割

实现字符串的分割的方法有很多种，常用的是 split ，StringTokenizer ，indexOf 和 substring 的配合，以及一些开源工具类，如：StringUtils。它们各有优缺。

```
@Test
public void test(){
    //数据初始化
    StringBuffer sb = new StringBuffer();
    for(int i=0;i<10000;i++){
        sb.append(i).append(";");
    }
    String originStr = sb.toString();
    //第一种分隔字符方法
    long startTime = System.nanoTime();

    String[] splitArray =  originStr.split(";");
    for(int i=0,len = splitArray.length;i<len;i++){
        String temp = splitArray[i];
    }
    long endTime = System.nanoTime();
    System.out.println("the cost of split is :" + (endTime - startTime));
    //第二种分隔字符方法
    System.out.println("--------------------------------------------");
    originStr = sb.toString();
    startTime = System.nanoTime();
    StringTokenizer st = new StringTokenizer(originStr,";");
    while(st.hasMoreTokens()){
        st.nextToken();
    }
    endTime = System.nanoTime();
    System.out.println("the cost of stringTokenizer is :" + (endTime - startTime));
    //第三种分隔字符的方法
    System.out.println("--------------------------------------------");
    originStr = sb.toString();
    startTime = System.nanoTime();
    while (true){
        int index = originStr.indexOf(";");
        if(index < 0) break;
        String origin = originStr.substring(0,index);
        originStr = originStr.substring(index + 1);
    }
    endTime = System.nanoTime();
    System.out.println("the cost of indexOf is :" + (endTime - startTime));

    //第四种分隔字符的方法
    System.out.println("--------------------------------------------");
    originStr = sb.toString();
    startTime = System.nanoTime();
    String[] utilSplit = StringUtils.split(originStr,';');
    for(int i=0,len = utilSplit.length;i<len;i++){
        String temp = utilSplit[i];
    }
    endTime = System.nanoTime();
    System.out.println("the cost of StringUtils.split is :" + (endTime - startTime));

}
```

运行结果：

```
the cost of split is :35710479
--------------------------------------------
the cost of stringTokenizer is :11992643
--------------------------------------------
the cost of indexOf is :323050471
--------------------------------------------
the cost of StringUtils.split is :59026333
```

从上面例子可以看出，字符分割的性能，由高到低的排序为：StringTokenizer &gt; split ，StringUtils.split &gt; indexOf 。有些书籍写着 indexOf 的性能是最高的，但是按照我的测试，index的性能是最差的。但是事物都有两面性，从上面的例子也可以看出，虽然 StringTokenizer 的性能高，但是代码量多，可读性差，而 split 代码相对就整洁多了。

#### **切勿把异常放置在循环体内** {#切勿把异常放置在循环体内}

try-catch语句本身性能不高，如果再放到循环体中，无非是雪上加霜。因此在开发中，我们要极力避免。

例：

```
for(int i=0;i<10;i++){
    try{

    }catch (Exception e){

    }
}
```

正确做法：

```
try{
    for(int i=0;i<10;i++){

    }
}catch (Exception e){

}
```

综上所述：不要再循环体内执行复制，耗时的操作。

尽量缩小锁的范围

锁优化的思路和方法总结一下，有以下几种。

减少锁持有时间（尽量缩小锁的范围）

减小锁粒度

锁分离

锁粗化

锁消除

我们应该确保我们只在必要的地方加锁，将锁从方法声明移到方法体中会延迟锁的加载，进而降低了锁竞争的可能性。先看下面的实例：

```
class SynObj {

    //方法锁/或者对象锁
    public synchronized void methodA() {
        System.out.println("methodA.....");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public  void methodB() {
        //对代码块进行锁，降低锁的竞争
        synchronized(this) {
            System.out.println("methodB.....");
        }
    }

    public void methodC() {
        String str = "sss";
        //这里锁的是 str 这个对象，而不是 SynObj 对象
        synchronized (str) {
            System.out.println("methodC.....");
        }
    }
}

/**
 * Created by Ay on 2017/3/26.
 */
public class AyTest {

    public static void main(String[] args) {
        final SynObj obj = new SynObj();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                obj.methodA();
            }
        });
        t1.start();

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                obj.methodB();
            }
        });
        t2.start();

        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() {
                obj.methodC();
            }
        });
        t3.start();
    }
}
```

打印结果:

```
methodA.....
methodC.....
//methodB会隔一段时间才会打印出来
methodB.....
```

总结：因为，一个线程访问了 synchronized 同步代码块中的代码，另一个线程不可以访问该对象的任何同步代码块，但可以访问非同步代码块。所有缩小锁的范围可以在一定程度上提高代码性能。

#### **锁分离** {#锁分离}

* 最常见的锁分离就是读写锁ReadWriteLock，根据功能进行分离成读锁和写锁，这样读读不互斥，读写互斥，写写互斥，即保证了线程安全，又提高了性能。

还有就是网上一个高手写的一个例子：

```
public class Grocery {
    private final ArrayList fruits = new ArrayList();
    private final ArrayList vegetables = new ArrayList();
    //对象锁，不好，效率低
    public synchronized void addFruit(int index, String fruit) {
        fruits.add(index, fruit);
    }
    //对象锁，不好，效率低
    public synchronized void removeFruit(int index) {
        fruits.remove(index);
    }
    //对象锁，不好，效率低
    public synchronized void addVegetable(int index, String vegetable) {
        vegetables.add(index, vegetable);
    }
    //对象锁，不好，效率低
    public synchronized void removeVegetable(int index) {
        vegetables.remove(index);
    }
}
```

优化后：

```
public class Grocery {
    private final ArrayList fruits = new ArrayList();
    private final ArrayList vegetables = new ArrayList();
    public void addFruit(int index, String fruit) {
        //水果锁 
        synchronized(fruits) fruits.add(index, fruit);
    }
    public void removeFruit(int index) {
        //水果锁 
        synchronized(fruits) {fruits.remove(index);}
    }
    public void addVegetable(int index, String vegetable) {
        //蔬菜锁
        synchronized(vegetables) vegetables.add(index, vegetable);
    }
    public void removeVegetable(int index) {
        //蔬菜锁
        synchronized(vegetables) vegetables.remove(index);
    }
}
```

#### **文件导入导出注意使用缓存流** {#文件导入导出注意使用缓存流}

#### **批量插入数据性能优化** {#批量插入数据性能优化}

##### **1.1 问题一** {#11-问题一}

直接批量保存3万多条数据。

```
List<PcsTestcase> pcsTestcases = new ArrayList<>();
// ......
//直接调用批量保存  
this.batchCreate(pcsTestcases);
```

#### **1.2 问题二** {#12-问题二}

批量保存时，利用UUID生成工具，给主键设置Id。找出Hibernate的先查询后更新的机制触发，造成不必要的查询损耗。

```
List<PcsTestcase> pcsTestcases = new ArrayList<>();
PcsTestcase pcsTestcase = null;
for (int j = sheet.getFirstRowNum() + 1,len = sheet.getLastRowNum(); j <= len;j++) {
    Row row = sheet.getRow(j);
    if (row == null) continue;
    pcsTestcase = new PcsTestcase();
    //看这里，重要：这里在插入数据时，设置主键Id
    pcsTestcase.setId(UUIDUtils.generate());
    pcsTestcase.setPmMilestoneId(pcsMainTask.getId());
}
```

#### **1.1 问题一解决方法** {#11-问题一解决方法}

对于问题二，我们可以把所有数据，每500条进行一次批量保存操作，速度会比一次性批量保存好。具体如下：

```
if(j % 500 == 0 || j == len){
    this.batchCreate(pcsTestcases);
    pcsTestcases = new ArrayList<>();
}
```

##### **1.2 问题二解决方法** {#12-问题二解决方法}

对于问题三，由于Hibernate在进行插入时，会判断数据是进行插入还是进行更新。**如果模型的主键不为空，查询数据后，再进行更新数据，否则，进行插入数据操作。**因此，我们在进行插入操作时候，不要设置模型的主键，可以避免不必要查询消耗。

```
pcsTestcase.setId(UUIDUtils.generate());
```

### **业务层面** {#业务层面}

* 减少前端请求数
* 过度复用方法带来的性能问题
* 后端如果需要一次性加载数据，防止多次请求数据库

### **数据库层面** {#数据库层面}

#### **SQL语句大小写规范** {#sql语句大小写规范}

我们在写SQL的时候，通常会出现大小写混用的情况。如下：

```
select * FROM pm_testcase pt where pt.Name = 'ay'
```

正确的做法是SQL语句全部大写或者全部小写。如下：

```
-- 全部小写
select * from pm_testcase pt where pt.name = 'ay'

-- 全部大写
SELECT * FROM PM_TESTCASE PT WHERE PT.NAME = 'ay'
```

#### **PostgreSQL执行计划** {#postgresql执行计划}

PostgreSQL的执行计划，做为数据库性能调优的利器，有必要在开头简单的介绍下。

```
explain analyse select * from pm_testcase pt
--执行计划
Seq Scan on pm_testcase pt  (cost=0.00..5237.11 rows=60011 width=2020) (actual time=37.347..435.601 rows=60012 loops=1)
Planning time: 0.426 ms
Execution time: 438.442 ms
```

cost说明：

第一个数字0.00表示启动cost，这是执行到返回第一行时需要的cost值。

第二个数字4621.00表示执行整个SQL的cost

通过查看执行计划，我们就能够找到SQL中的哪部分比较慢，或者说花费时间多。然后重点分析哪部分的逻辑，比如减少循环查询，或者强制改变执行计划。

更多执行计划 Explain，可网上搜索。

## 建立索引避免全表扫描

首先，在数据库里有一张表 pm\_testcase，里面有150万条数据。

如下SQL，我们利用执行计划，对创建时间（created\_time）进行排序，输出执行计划结果。

```
程序清单 2-1

explain 
select * from pm_testcase pt
order by pt.created_time desc

--Sort  (cost=4103259.72..4107084.44 rows=1529885 width=1920)
--Sort Key: created_time
--->  Seq Scan on pm_testcase pt  (cost=0.00..134087.85 rows=1529885 width=1920)
```

cost=说明：

第一个数字4103259.72表示启动cost，这是执行到返回第一行时需要的cost值。

第二个数字4107084.44表示执行整个SQL的cost。

该语句总共耗时 4107084.44

这里我们创建 created\_time 索引，对相同语句执行 程序清单 2-1 的SQL，得到的执行计划结果为：

```
Index Scan Backward using idx_create_time on pm_testcase pt  (cost=0.43..384739.28 rows=1530024 width=1920)
```

很明显，执行整个SQL的 cost 由 4107084.44 减少到 384739.28

因此，为了避免全表扫描，建议在考虑在 where 及 order by 涉及的列上建立索引。

#### **防止索引失效** {#防止索引失效}

我们应尽量避免在 where 子句中使用 != 或 &lt;&gt; 操作符，否则引擎将放弃使用索引而进行全表扫描。

如下例子，我们在 pm\_testcase 的 code 上添加了索引：

```
explain select pt.code from pm_testcase pt
where pt.code != 'case005510'

--执行计划，Seq Scan 全表扫描
Seq Scan on pm_testcase pt  (cost=0.00..137914.30 rows=1529973 width=11)

explain select pt.code from pm_testcase pt
where pt.code = 'case005510'

--执行计划，Bitmap Heap Scan 索引扫描
Bitmap Heap Scan on pm_testcase pt  (cost=4.82..206.29 rows=51 width=11)
```

通过上面的例子可以看出，!= 操作符使得索引失效。

## 避免建立太多的索引

索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 和 update 的效率，因为 insert 或 update 时有可能会重建索引，所以视具体情况而定。一个表的索引数最好不要超过7个，若太多则应考虑一些不常使用到的列上建的索引是否有必要.

## 关于查询效率的几点建议

尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。

尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为变长字段存储空间小，对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

最好不要给数据库留NULL，尽可能的使用 NOT NULL填充数据库。备注、描述、评论之类的可以设置为 NULL。其他的，最好不要使用NULL。

任何地方都不要使用 select \* from t ，用具体的字段列表代替 \* ，不要返回用不到的任何字段。

应尽量避免在 where 子句中使用 or 来连接条件,可以考虑使用 union 代替

in 和 not in 也要慎用。对于连续的数值，能用 between 就不要用 in，exists 代替 in

尽量避免在 where 子句中对字段进行表达式操作和函数操作

## 在Join表的时候字段使用相同类型，并将其索引

如果你的应用程序有很多JOIN查询，你应该确认两个表中Join的字段是被建过索引的。这样，SQL内部会启动为你优化Join的SQL语句的机制。而且，这些被用来Join的字段，应该是相同的类型的。例如：如果你要把 DECIMAL 字段和一个 INT 字段 Join 在一起，SQL 就无法使用它们的索引。对于那些STRING 类型，还需要有相同的字符集才行。（两个表的字符集有可能不一样）程序员站

#### **优化子查询** {#优化子查询}

子查询很灵活可以极大的节省查询的步骤，但是子查询的执行效率不高。执行子查询时数据库需要为内部嵌套的语句查询的结果建立一个临时表，然后再使用临时表中的数据进行查询。查询完成后再删除这个临时表，所以子查询的速度会慢一点。

我们可以使用join语句来替换掉子查询，来提高效率。join语句不需要建立临时表，所以其查询速度会优于子查询。大部分的不是很复杂的子查询都可以替换成join语句。

## 服务器层面

服务器的调优，就得根据客户提供的真实环境的配置。如服务器是几核几个CPU等等。服务器的硬件指标确定下来后，根据指标调整Tomcat，JDK，数据库，Apatch等配置参数。让整个环境达到最优的效果。这块工作一般不是开发人员进行的。但是我们要了解清楚一些配置参数

## Tomcat && JDK

**tomcat 配置**

**JDK垃圾回收机制**

**垃圾回收机制算法选择**

**JVM内存模型**

## Postgresql数据库配置

### Linux服务器

### 输出系统日志最后10行 dmesg \| tail

```
ubuntu@ubuntu:~$ dmesg | tail
[38060.138072] e1000: eno16777736 NIC Link is Down
[38068.362442] e1000: eno16777736 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
[38070.366445] e1000: eno16777736 NIC Link is Down
[38076.376947] e1000: eno16777736 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
[38084.386812] e1000: eno16777736 NIC Link is Down
[38090.411818] e1000: eno16777736 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
[38480.597723] e1000: eno16777736 NIC Link is Down
[38495.064487] e1000: eno16777736 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
[38607.910407] IPv6: ADDRCONF(NETDEV_UP): eno16777736: link is not ready
[38607.978329] e1000: eno16777736 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
```

该命令会输出系统日志的最后10行。这些日志可以帮助排查性能问题。

##### **top命令** {#top命令}

top命令是进行性能分析最常使用的命令，也是最重要的命令。每个参数代表什么意思，都必须非常清楚。

```
top - 07:01:15 up 10:57,  3 users,  load average: 0.00, 0.04, 0.13
Tasks: 238 total,   1 running, 237 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.4 us,  3.8 sy,  0.0 ni, 92.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2040024 total,  2020312 used,    19712 free,    11220 buffers
KiB Swap:  3142652 total,   927204 used,  2215448 free.   121276 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                   
 6844 root      20   0  333020  20520   3600 S   6.0  1.0  29:48.44 Xorg                                                                      
61687 ubuntu    20   0 1635056  43716  18108 S   3.6  2.1   5:00.27 compiz                                                                    
 5444 ubuntu    20   0 3765292 875688  10020 S   2.7 42.9  42:13.69 java                                                                      
 6788 root      20   0  293028   9284   1112 S   2.3  0.5   0:51.92 dockerd                                                                   
 5175 ubuntu    20   0  578736  22496  14888 S   1.7  1.1   0:04.60 gnome-terminal-                                                           
   27 root      39  19       0      0      0 S   0.7  0.0   0:09.02 khugepaged                                                                
 7932 ubuntu    20   0 3060636  16560 
```




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




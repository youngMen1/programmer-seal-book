### **前言** {#前言}

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
————————————————
版权声明：本文为CSDN博主「阿_毅」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/huangwenyi1010/article/details/72673447
```




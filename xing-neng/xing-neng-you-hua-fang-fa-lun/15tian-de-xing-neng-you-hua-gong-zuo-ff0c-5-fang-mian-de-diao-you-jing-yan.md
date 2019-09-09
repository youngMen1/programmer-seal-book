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




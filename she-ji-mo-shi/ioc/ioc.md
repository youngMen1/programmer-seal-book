作者：Mingqi

  


链接：https://www.zhihu.com/question/23277575/answer/169698662

  


来源：知乎

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

  


  


要了解**控制反转\( Inversion of Control \)**, 我觉得有必要先了解软件设计的一个重要思想：**依赖倒置原则（Dependency Inversion Principle ）**。

&lt;

img src="https://pic1.zhimg.com/50/v2-d53c75e91d959acbb0d95a835212ada5\_hd.jpg" data-caption="" data-rawwidth="600" data-rawheight="61" class="origin\_image zh-lightbox-thumb" width="600" data-original="https://pic1.zhimg.com/v2-d53c75e91d959acbb0d95a835212ada5\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-d53c75e91d959acbb0d95a835212ada5_hd.jpg)

**什么是依赖倒置原则？**假设我们设计一辆汽车：先设计轮子，然后根据轮子大小设计底盘，接着根据底盘设计车身，最后根据车身设计好整个汽车。这里就出现了一个“依赖”关系：汽车依赖车身，车身依赖底盘，底盘依赖轮子。

&lt;

img src="https://pic4.zhimg.com/50/v2-c68248bb5d9b4d64d22600571e996446\_hd.jpg" data-caption="" data-rawwidth="1562" data-rawheight="186" class="origin\_image zh-lightbox-thumb" width="1562" data-original="https://pic4.zhimg.com/v2-c68248bb5d9b4d64d22600571e996446\_r.jpg"/

&gt;

![](https://pic4.zhimg.com/80/v2-c68248bb5d9b4d64d22600571e996446_hd.jpg)

这样的设计看起来没问题，但是可维护性却很低。假设设计完工之后，上司却突然说根据市场需求的变动，要我们把车子的轮子设计都改大一码。这下我们就蛋疼了：因为我们是根据轮子的尺寸设计的底盘，轮子的尺寸一改，底盘的设计就得修改；同样因为我们是根据底盘设计的车身，那么车身也得改，同理汽车设计也得改——整个设计几乎都得改！

我们现在换一种思路。我们先设计汽车的大概样子，然后根据汽车的样子来设计车身，根据车身来设计底盘，最后根据底盘来设计轮子。这时候，依赖关系就倒置过来了：轮子依赖底盘， 底盘依赖车身， 车身依赖汽车。

&lt;

img src="https://pic1.zhimg.com/50/v2-e64bf72c5c04412f626b21753aa9e1a1\_hd.jpg" data-caption="" data-rawwidth="1504" data-rawheight="190" class="origin\_image zh-lightbox-thumb" width="1504" data-original="https://pic1.zhimg.com/v2-e64bf72c5c04412f626b21753aa9e1a1\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-e64bf72c5c04412f626b21753aa9e1a1_hd.jpg)

这时候，上司再说要改动轮子的设计，我们就只需要改动轮子的设计，而不需要动底盘，车身，汽车的设计了。

这就是依赖倒置原则——把原本的高层建筑依赖底层建筑“倒置”过来，变成底层建筑依赖高层建筑。高层建筑决定需要什么，底层去实现这样的需求，但是高层并不用管底层是怎么实现的。这样就不会出现前面的“牵一发动全身”的情况。

&lt;

img src="https://pic1.zhimg.com/50/v2-d53c75e91d959acbb0d95a835212ada5\_hd.jpg" data-caption="" data-rawwidth="600" data-rawheight="61" class="origin\_image zh-lightbox-thumb" width="600" data-original="https://pic1.zhimg.com/v2-d53c75e91d959acbb0d95a835212ada5\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-d53c75e91d959acbb0d95a835212ada5_hd.jpg)

**控制反转（Inversion of Control）** 就是依赖倒置原则的一种代码设计的思路。具体采用的方法就是所谓的**依赖注入（Dependency Injection）**。其实这些概念初次接触都会感到云里雾里的。说穿了，这几种概念的关系大概如下：

&lt;

img src="https://pic1.zhimg.com/50/v2-ee924f8693cff51785ad6637ac5b21c1\_hd.jpg" data-caption="" data-rawwidth="1398" data-rawheight="630" class="origin\_image zh-lightbox-thumb" width="1398" data-original="https://pic1.zhimg.com/v2-ee924f8693cff51785ad6637ac5b21c1\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-ee924f8693cff51785ad6637ac5b21c1_hd.jpg)

为了理解这几个概念，我们还是用上面汽车的例子。只不过这次换成代码。我们先定义四个Class，车，车身，底盘，轮胎。然后初始化这辆车，最后跑这辆车。代码结构如下：

&lt;

img src="https://pic3.zhimg.com/50/v2-8ec294de7d0f9013788e3fb5c76069ef\_hd.jpg" data-caption="" data-rawwidth="512" data-rawheight="717" class="origin\_image zh-lightbox-thumb" width="512" data-original="https://pic3.zhimg.com/v2-8ec294de7d0f9013788e3fb5c76069ef\_r.jpg"/

&gt;

![](https://pic3.zhimg.com/80/v2-8ec294de7d0f9013788e3fb5c76069ef_hd.jpg)

这样，就相当于上面第一个例子，上层建筑依赖下层建筑——每一个类的构造函数都直接调用了底层代码的构造函数。假设我们需要改动一下轮胎（Tire）类，把它的尺寸变成动态的，而不是一直都是30。我们需要这样改：

&lt;

img src="https://pic4.zhimg.com/50/v2-64e8b19eeb70d9cf87c27fe4c5c0fc81\_hd.jpg" data-caption="" data-rawwidth="534" data-rawheight="154" class="origin\_image zh-lightbox-thumb" width="534" data-original="https://pic4.zhimg.com/v2-64e8b19eeb70d9cf87c27fe4c5c0fc81\_r.jpg"/

&gt;

![](https://pic4.zhimg.com/80/v2-64e8b19eeb70d9cf87c27fe4c5c0fc81_hd.jpg)

由于我们修改了轮胎的定义，为了让整个程序正常运行，我们需要做以下改动：

&lt;

img src="https://pic3.zhimg.com/50/v2-82e0c12a1b26f7979ed9241e169affda\_hd.jpg" data-caption="" data-rawwidth="1186" data-rawheight="1452" class="origin\_image zh-lightbox-thumb" width="1186" data-original="https://pic3.zhimg.com/v2-82e0c12a1b26f7979ed9241e169affda\_r.jpg"/

&gt;

![](https://pic3.zhimg.com/80/v2-82e0c12a1b26f7979ed9241e169affda_hd.jpg)

由此我们可以看到，仅仅是为了修改轮胎的构造函数，这种设计却需要**修改整个上层所有类的构造函数**！在软件工程中，**这样的设计几乎是不可维护的**——在实际工程项目中，有的类可能会是几千个类的底层，如果每次修改这个类，我们都要修改所有以它作为依赖的类，那软件的维护成本就太高了。

所以我们需要进行控制反转（IoC），及上层控制下层，而不是下层控制着上层。我们用依赖注入（Dependency Injection）这种方式来实现控制反转。**所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制**”。这里我们用**构造方法传递的依赖注入方式**重新写车类的定义：

&lt;

img src="https://pic1.zhimg.com/50/v2-c920a0540ce0651003a5326f6ef9891d\_hd.jpg" data-caption="" data-rawwidth="1338" data-rawheight="1424" class="origin\_image zh-lightbox-thumb" width="1338" data-original="https://pic1.zhimg.com/v2-c920a0540ce0651003a5326f6ef9891d\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-c920a0540ce0651003a5326f6ef9891d_hd.jpg)

这里我们再把轮胎尺寸变成动态的，同样为了让整个系统顺利运行，我们需要做如下修改：

&lt;

img src="https://pic4.zhimg.com/50/v2-99ad2cd809fcb86dd791ff7f65fb1779\_hd.jpg" data-caption="" data-rawwidth="1344" data-rawheight="1424" class="origin\_image zh-lightbox-thumb" width="1344" data-original="https://pic4.zhimg.com/v2-99ad2cd809fcb86dd791ff7f65fb1779\_r.jpg"/

&gt;

![](https://pic4.zhimg.com/80/v2-99ad2cd809fcb86dd791ff7f65fb1779_hd.jpg)

看到没？这里**我只需要修改轮胎类就行了，不用修改其他任何上层类。**这显然是更容易维护的代码。不仅如此，在实际的工程中，这种设计模式还有利于**不同组的协同合作和单元测试：**比如开发这四个类的分别是四个不同的组，那么只要定义好了接口，四个不同的组可以同时进行开发而不相互受限制；而对于单元测试，如果我们要写Car类的单元测试，就只需要Mock一下Framework类传入Car就行了，而不用把Framework, Bottom, Tire全部new一遍再来构造Car。

这里我们是采用的**构造函数传入**的方式进行的依赖注入。其实还有另外两种方法：**Setter传递**和**接口传递**。这里就不多讲了，核心思路都是一样的，都是为了实现**控制反转**。

&lt;

img src="https://pic1.zhimg.com/50/v2-861683acac47577c81f2b7493dd05649\_hd.jpg" data-caption="" data-rawwidth="924" data-rawheight="298" class="origin\_image zh-lightbox-thumb" width="924" data-original="https://pic1.zhimg.com/v2-861683acac47577c81f2b7493dd05649\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-861683acac47577c81f2b7493dd05649_hd.jpg)

  


&lt;

img src="https://pic1.zhimg.com/50/v2-d53c75e91d959acbb0d95a835212ada5\_hd.jpg" data-caption="" data-rawwidth="600" data-rawheight="61" class="origin\_image zh-lightbox-thumb" width="600" data-original="https://pic1.zhimg.com/v2-d53c75e91d959acbb0d95a835212ada5\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-d53c75e91d959acbb0d95a835212ada5_hd.jpg)

看到这里你应该能理解什么控制反转和依赖注入了。那什么是**控制反转容器\(IoC Container\)**呢？其实上面的例子中，对车类进行初始化的那段代码发生的地方，就是控制反转容器。

&lt;

img src="https://pic4.zhimg.com/50/v2-c845802f9187953ed576e0555f76da42\_hd.jpg" data-caption="" data-rawwidth="1422" data-rawheight="628" class="origin\_image zh-lightbox-thumb" width="1422" data-original="https://pic4.zhimg.com/v2-c845802f9187953ed576e0555f76da42\_r.jpg"/

&gt;

![](https://pic4.zhimg.com/80/v2-c845802f9187953ed576e0555f76da42_hd.jpg)

显然你也应该观察到了，因为采用了依赖注入，在初始化的过程中就不可避免的会写大量的new。这里IoC容器就解决了这个问题。**这个容器可以自动对你的代码进行初始化，你只需要维护一个Configuration（可以是xml可以是一段代码），而不用每次初始化一辆车都要亲手去写那一大段初始化的代码**。这是引入IoC Container的第一个好处。

IoC Container的第二个好处是：**我们在创建实例的时候不需要了解其中的细节。**在上面的例子中，我们自己手动创建一个车instance时候，是从底层往上层new的：

&lt;

img src="https://pic2.zhimg.com/50/v2-555b2be7d76e78511a6d6fed3304927f\_hd.jpg" data-caption="" data-rawwidth="2430" data-rawheight="168" class="origin\_image zh-lightbox-thumb" width="2430" data-original="https://pic2.zhimg.com/v2-555b2be7d76e78511a6d6fed3304927f\_r.jpg"/

&gt;

![](https://pic2.zhimg.com/80/v2-555b2be7d76e78511a6d6fed3304927f_hd.jpg)

这个过程中，我们需要了解整个Car/Framework/Bottom/Tire类构造函数是怎么定义的，才能一步一步new/注入。

而IoC Container在进行这个工作的时候是反过来的，它先从最上层开始往下找依赖关系，到达最底层之后再往上一步一步new（有点像深度优先遍历）：

&lt;

img src="https://pic3.zhimg.com/50/v2-24a96669241e81439c636e83976ba152\_hd.jpg" data-caption="" data-rawwidth="2522" data-rawheight="354" class="origin\_image zh-lightbox-thumb" width="2522" data-original="https://pic3.zhimg.com/v2-24a96669241e81439c636e83976ba152\_r.jpg"/

&gt;

![](https://pic3.zhimg.com/80/v2-24a96669241e81439c636e83976ba152_hd.jpg)

这里IoC Container可以直接隐藏具体的创建实例的细节，在我们来看它就像一个工厂：

&lt;

img src="https://pic1.zhimg.com/50/v2-5ca61395f37cef73c7bbe7808f9ea219\_hd.jpg" data-caption="" data-rawwidth="2448" data-rawheight="524" class="origin\_image zh-lightbox-thumb" width="2448" data-original="https://pic1.zhimg.com/v2-5ca61395f37cef73c7bbe7808f9ea219\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-5ca61395f37cef73c7bbe7808f9ea219_hd.jpg)

我们就像是工厂的客户。我们只需要向工厂请求一个Car实例，然后它就给我们按照Config创建了一个Car实例。我们完全不用管这个Car实例是怎么一步一步被创建出来。

实际项目中，有的Service Class可能是十年前写的，有几百个类作为它的底层。假设我们新写的一个API需要实例化这个Service，我们总不可能回头去搞清楚这几百个类的构造函数吧？IoC Container的这个特性就很完美的解决了这类问题——**因为这个架构要求你在写class的时候需要写相应的Config文件，所以你要初始化很久以前的Service类的时候，前人都已经写好了Config文件，你直接在需要用的地方注入这个Service就可以了**。这大大增加了项目的可维护性且降低了开发难度。

&lt;

img src="https://pic1.zhimg.com/50/v2-d53c75e91d959acbb0d95a835212ada5\_hd.jpg" data-caption="" data-rawwidth="600" data-rawheight="61" class="origin\_image zh-lightbox-thumb" width="600" data-original="https://pic1.zhimg.com/v2-d53c75e91d959acbb0d95a835212ada5\_r.jpg"/

&gt;

![](https://pic1.zhimg.com/80/v2-d53c75e91d959acbb0d95a835212ada5_hd.jpg)

这里只是很粗略的讲了一下我自己对IoC和DI的理解。主要的目的是在于**最大限度避免晦涩难懂的专业词汇，用尽量简洁，通俗，直观的例子**来解释这些概念。如果让大家能有一个类似“哦！原来就是这么个玩意嘛！”的印象，我觉得就OK了。想要深入了解的话，可以上网查阅一些更权威的资料。这里推荐一下 [Dependency injection](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Dependency_injection) 和 [Inversion of Control Containers and the Dependency Injection pattern](https://link.zhihu.com/?target=https%3A//martinfowler.com/articles/injection.html) 这两篇文章，讲的很好很详细。


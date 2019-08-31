## Jetty 的基本架构 {#major1}

Jetty 目前的是一个比较被看好的 Servlet 引擎，它的架构比较简单，也是一个可扩展性和非常灵活的应用服务器，它有一个基本数据模型，这个数据模型就是 Handler，所有可以被扩展的组件都可以作为一个 Handler，添加到 Server 中，Jetty 就是帮你管理这些 Handler。

### Jetty 的基本架构 {#minor1.1}

下图是 Jetty 的基本架构图，整个 Jetty 的核心组件由 Server 和 Connector 两个组件构成，整个 Server 组件是基于 Handler 容器工作的，它类似与 Tomcat 的 Container 容器，Jetty 与 Tomcat 的比较在后面详细介绍。Jetty 中另外一个比不可少的组件是 Connector，它负责接受客户端的连接请求，并将请求分配给一个处理队列去执行。

##### 图 1. Jetty 的基本架构 {#fig1}

image003.png

Jetty 中还有一些可有可无的组件，我们可以在它上做扩展。如 JMX，我们可以定义一些 Mbean 把它加到 Server 中，当 Server 启动的时候，这些 Bean 就会一起工作。

##### 图 2. Jetty 的主要组件的类图 {#fig2}

image005.jpg

从上图可以看出整个 Jetty 的核心是围绕着 Server 类来构建，Server 类继承了 Handler，关联了 Connector 和 Container。Container 是管理 Mbean 的容器。Jetty 的 Server 的扩展主要是实现一个个 Handler 并将 Handler 加到 Server 中，Server 中提供了调用这些 Handler 的访问规则。

整个 Jetty 的所有组件的生命周期管理是基于观察者模板设计，它和 Tomcat 的管理是类似的。下面是 LifeCycle 的类关系图

image007.jpg

##### 每个组件都会持有一个观察者（在这里是 Listener 类，这个类通常对应到观察者模式中常用的 Observer 角色，关于观察者模式可以参考[《](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[Tomcat](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[系统架构与设计模式，第](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[2](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[部分](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[:](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)[设计模式分析》](http://www.ibm.com/developerworks/cn/java/j-lo-tomcat2/index.html)一文中关于观察者模式的讲解）集合，当 start、fail 或 stop 等事件触发时，这些 Listener 将会被调用，这是最简单的一种设计方式，相比 Tomcat 的 LifeCycle 要简单的多。 {#fig3}

### Handler 的体系结构 {#minor1.2}

前面所述 Jetty 主要是基于 Handler 来设计的，Handler 的体系结构影响着整个 Jetty 的方方面面。下面总结了一下 Handler 的种类及作用：

##### 图 3. Handler 的体系结构（[查看大图](https://www.ibm.com/developerworks/cn/java/j-lo-jetty/image008.png)） {#fig4}

##### image009.jpg {#fig4}

Jetty 主要提供了两种 Handler 类型，一种是 HandlerWrapper，它可以将一个 Handler 委托给另外一个类去执行，如我们要将一个 Handler 加到 Jetty 中，那么就必须将这个 Handler 委托给 Server 去调用。配合 ScopeHandler 类我们可以拦截 Handler 的执行，在调用 Handler 之前或之后，可以做一些另外的事情，类似于 Tomcat 中的 Valve；另外一个 Handler 类型是 HandlerCollection，这个 Handler 类可以将多个 Handler 组装在一起，构成一个 Handler 链，方便我们做扩展。

## Jetty 的启动过程 {#major2}

Jetty 的入口是 Server 类，Server 类启动完成了，就代表 Jetty 能为你提供服务了。它到底能提供哪些服务，就要看 Server 类启动时都调用了其它组件的 start 方法。从 Jetty 的配置文件我们可以发现，配置 Jetty 的过程就是将那些类配置到 Server 的过程。下面是 Jetty 的启动时序图：

##### 图 4. Jetty 的启动流程 {#fig5}




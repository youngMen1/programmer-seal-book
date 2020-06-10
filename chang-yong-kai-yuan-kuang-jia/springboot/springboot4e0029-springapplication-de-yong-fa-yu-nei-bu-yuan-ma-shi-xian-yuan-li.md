# 1.SpringApplication的用法与内部源码实现原理

### 1.1.概述

* 在基于SpringBoot的web应用中，通常使用一个带有main方法的类，通过命令行执行main方法来启动整个应用。而在main方法中是使用SpringApplication.run这个静态方法或者创建SpringApplication对象，执行成员方法run，以该main方法所在的类作为参数的方式启动的。

* main方法所在的类是一个基于Spring的注解，如@Configuration，@ComponentScan等，的配置类。






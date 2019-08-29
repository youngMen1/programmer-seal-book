## 1. final的简介

final可以修饰\*\***变量，方法和类**\*\*，用于表示所修饰的内容一旦赋值之后就不会再被改变，比如String类就是一个final类型的类。即使能够知道final具体的使用方法，我想对\*\***final在多线程中存在的重排序问题**\*\*也很容易忽略，希望能够一起做下探讨。

## 2. final的具体使用场景

final能够修饰变量，方法和类，也就是final使用范围基本涵盖了java每个地方，下面就分别以锁修饰的位置：变量，方法和类分别来说一说。

### 2.1 变量

在java中变量，可以分为\***\*成员变量**\*\***以及方法**\*\***局部变量**\*\*。因此也是按照这种方式依次来说，以避免漏掉任何一个死角。

#### 2.1.1 final成员变量

通常每个类中的成员变量可以分为\*\***类变量（static修饰的变量）以及实例变量**\*\*。针对这两种类型的变量赋初值的时机是不同的，类变量可以在声明变量的时候直接赋初值或者在静态代码块中给类变量赋初值。而实例变量可以在声明变量的时候给实例变量赋初值，在非静态初始化块中以及构造器中赋初值。类变量有\*\***两个时机赋初值**\*\*，而实例变量则可以有\*\***三个时机赋初值**\*\*。当final变量未初始化时系统不会进行隐式初始化，会出现报错。这样说起来还是比较抽象，下面用具体的代码来演示。（代码涵盖了final修饰变量所有的可能情况，耐心看下去会有收获的:\) ）

![](/assets/final修饰成员变量.png)

!\[final修饰成员变量\]\([http://upload-images.jianshu.io/upload\_images/2615789-768017317b5fab78.png?imageMogr2/auto-](http://upload-images.jianshu.io/upload_images/2615789-768017317b5fab78.png?imageMogr2/auto-)

orient/strip%7CimageView2/2/w/1240\)

看上面的图片已经将每种情况整理出来了，这里用截图的方式也是觉得在IDE出现红色出错的标记更能清晰的说明情况。现在我们来将这几种

情况归纳整理一下：

1. \*\***类变量**\*\*：必须要在\*\***静态初始化块**\*\*中指定初始值或者\*\*声明该类变量时\*\*指定初始值，而且只能在这\*\***两个地方**\*\*之一进行指定；

2. \*\***实例变量**\*\*：必要要在\*\***非静态初始化块**\*\*，\*\*声明该实例变量\*\*或者在\*\*构造器中\*\*指定初始值，而且只能在这\*\***三个地方**\*\*进行指定。

#### 2.2.2 final局部变量

final局部变量由程序员进行显式初始化，如果final局部变量已经进行了初始化则后面就不能再次进行更改，如果final变量未进行初始化，可以进行赋值，\*\***当且仅有一次**\*\*赋值，一旦赋值之后再次赋值就会出错。下面用具体的代码演示final局部变量的情况：

![](/assets/final修饰局部变量.png)

!\[final修饰局部变量\]\([http://upload-images.jianshu.io/upload\_images/2615789-7077bdb169d4d1c3.png?imageMogr2/auto-](http://upload-images.jianshu.io/upload_images/2615789-7077bdb169d4d1c3.png?imageMogr2/auto-)

orient/strip%7CimageView2/2/w/1240\)

现在我们来换一个角度进行考虑，final修饰的是基本数据类型和引用类型有区别吗？

&gt; \*\***final基本数据类型  VS final引用数据类型**\*\*

通过上面的例子我们已经看出来，如果final修饰的是一个基本数据类型的数据，一旦赋值后就不能再次更改，那么，如果final是引用数据类型了？这个引用的对象能够改变吗？我们同样来看一段代码。

```
public class FinalExample {
        //在声明final实例成员变量时进行赋值
        private final static Person person = new Person(24, 170);
        public static void main(String[] args) {
            //对final引用数据类型person进行更改
            person.age = 22;
            System.out.println(person.toString());
        }
        static class Person {
            private int age;
            private int height;

            public Person(int age, int height) {
                this.age = age;
                this.height = height;
            }
            @Override
            public String toString() {
                return "Person{" +
                        "age=" + age +
                        ", height=" + height +
                        '}';
            }
        }
    }
```

当我们对final修饰的引用数据类型变量person的属性改成22，是可以成功操作的。通过这个实验我们就可以看出来\*\***当final修饰基本数据类型变量时，不能对基本数据类型变量重新赋值，因此基本数据类型变量不能被改变。而对于引用类型变量而言，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的地址不会发生改变，即一直引用这个对象，但这个对象属性是可以改变的**\*\*。

&gt; \*\***宏变量**\*\*

利用final变量的不可更改性，在满足一下三个条件时，该变量就会成为一个“宏变量”，即是一个常量。

1. 使用final修饰符修饰；

2. 在定义该final变量时就指定了初始值；

3. 该初始值在编译时就能够唯一指定。

注意：当程序中其他地方使用该宏变量的地方，编译器会直接替换成该变量的值

### 2.2 方法

&gt; \*\***重写？**\*\*

当父类的方法被final修饰的时候，子类不能重写父类的该方法，比如在Object中，getClass\(\)方法就是final的，我们就不能重写该方法，但是hashCode\(\)方法就不是被final所修饰的，我们就可以重写hashCode\(\)方法。我们还是来写一个例子来加深一下理解：

先定义一个父类，里面有final修饰的方法test\(\);

```
public class FinalExampleParent {
        public final void test() {
        }
    }
```

然后FinalExample继承该父类，当重写test\(\)方法时出现报错，如下图：

![](/assets/final方法不能重写.png)



!\[final方法不能重写\]\([http://upload-images.jianshu.io/upload\_images/2615789-5d831da449f512e9.png?imageMogr2/auto-](http://upload-images.jianshu.io/upload_images/2615789-5d831da449f512e9.png?imageMogr2/auto-)

orient/strip%7CimageView2/2/w/1240\)

通过这个现象我们就可以看出来\*\*被final修饰的方法不能够被子类所重写\*\*。


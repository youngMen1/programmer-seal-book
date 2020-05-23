# [泛型中K T V E ？ object等的含义](http://hollischuang.gitee.io/tobetopjavaer/#/basics/java-basic/k-t-v-e?id=%e6%b3%9b%e5%9e%8b%e4%b8%adk-t-v-e-%ef%bc%9f-object%e7%ad%89%e7%9a%84%e5%90%ab%e4%b9%89) {#泛型中k-t-v-e-？-object等的含义}

E - Element \(在集合中使用，因为集合中存放的是元素\)

T - Type（Java 类）

K - Key（键）

V - Value（值）

N - Number（数值类型）

？ - 表示不确定的java类型（无限制通配符类型）

S、U、V - 2nd、3rd、4th types

Object - 是所有类的根类，任何类的对象都可以设置给该Object引用变量，使用的时候可能需要类型强制转换，但是用使用了泛型T、E等这些标识符后，在实际用之前类型就已经确定了，不需要再进行类型强制转换。


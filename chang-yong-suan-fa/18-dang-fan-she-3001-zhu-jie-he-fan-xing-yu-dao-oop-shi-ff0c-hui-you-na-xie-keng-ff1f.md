# 18 | 当反射、注解和泛型遇到OOP时，会有哪些坑？

你好，我是朱晔。今天，我们聊聊 Java 高级特性的话题，看看反射、注解和泛型遇到重载和继承时可能会产生的坑。

你可能说，业务项目中几乎都是增删改查，用到反射、注解和泛型这些高级特性的机会少之又少，没啥好学的。但我要说的是，只有学好、用好这些高级特性，才能开发出更简洁易读的代码，而且几乎所有的框架都使用了这三大高级特性。比如，要减少重复代码，就得用到反射和注解（详见第 21 讲）。

如果你从来没用过反射、注解和泛型，可以先通过官网有一个大概了解：

Java Reflection API ：`https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html`

Reflection Tutorials:`https://docs.oracle.com/javase/tutorial/reflect/index.html`

Annotations: `https://docs.oracle.com/javase/8/docs/technotes/guides/language/annotations.html`

Lesson: Annotations:`https://docs.oracle.com/javase/tutorial/java/annotations/index.html`

Generics:`https://docs.oracle.com/javase/8/docs/technotes/guides/language/generics.html`

Lesson: Generics：`https://docs.oracle.com/javase/tutorial/java/generics/index.html`
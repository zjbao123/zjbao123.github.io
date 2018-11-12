---
title: java基础(一)对java的理解
date: 2018-11-08 17:35:00
tags: 
- java
categories: 总结
---


### Write once， run anywhere

“Write once， run anywhere” 说的是Java语言跨平台的特性，Java的跨平台特性与Java虚拟机的存在密不可分，可在不同的环境中运行。比如说Windows平台和Linux平台都有相应的JDK，安装好JDK后也就有了Java语言的运行环境。其实Java语言本身与其他的编程语言没有特别大的差异，并不是说Java语言可以跨平台，而是在不同的平台都有可以让Java语言运行的环境而已，所以才有了Java一次编译，到处运行这样的效果。有了编译器就屏蔽了不同机器语言的区别，有了系统API就屏蔽了不同硬件的区别，有了JVM就屏蔽了不同操作系统的区别，有了TCP/IP就屏蔽了不同系统之间通讯的差异。
<!-- more -->

### java语言特性

包括面向对象，泛型、反射、Lambda等语言特性；

#### Java类库

核心类库包括集合、IO/NIO等，网络，并发，安全等基础类库，其他第三方类库。

#### Java的类加载机制

常用版本JDK内嵌的Class-Loader，例如Bootstrap、Application和Extension Class-loader， 以及自定义Class-Loader等。

#### 类加载的大致过程

加载 \rightarrow 验证 \rightarrow 链接 \rightarrow 初始化

#### 垃圾收集的基本原理

最常见额垃圾收集器，如SerialGC、Parallel GC、CMS、G1等，适用于什么样的负载
还有运行时，动态编译，辅助功能JFR等

#### 工具类

`辅助工具`：jlink、jar、jdeps之类
`编译器`：javac，sjavac
`诊断工具`：jmap、jstack、jconsole、jhsdb，jcmd等进行监控诊断工具，程序的性能进行监控诊断，进行程序的诊断和调优工作

#### Java/JVM生态

Java EE，spring， Hadoop，Spark，Cassandra，ElasticSearch，Maven

### Java的编译时和运行时

Java分为编译时和运行时。
编译指的是生成`。class`字节码，而不是可以直接执行的机器码，通过字节码和Java虚拟机(JVM)这种跨平台的抽象来屏蔽了操作系统和硬件的细节，从而实现`write once， run anywhere。

在运行时，JVM会通过类加载器(class-Loader)加载字节码，解释或者编译执行。主流Java版本中，如JDK 8实际是解释和编译混合的一种模式，即混合模式(-Xmixed)。通常运行在server模式的JVM。会进行上万次调用以收集足够的信息来进行高效的编译，Client模式这个门限是1500次。Oracle Hotspot JVM 内置了两个不同的JIT complier，C1对应前面说的Client模式，适用于对于启动速度敏感的应用，比如普通的java桌面应用;C2对应server模式，他的优化是为长时间运行的服务器端的应用设计的。默认是采用所谓的分层编译(TieredCompilation)。

Java虚拟机启动时，可以指定不同的参数对运行模式进行选择。比如指定`-Xint`，就是告诉JVM只进行解释执行，不对代码进行编译，这种模式抛弃了JIT可能带来的性能优势。毕竟解释其是逐条读入，逐条运行的。与其相对应的，还有一个`-XComp`会导致JVM启动变慢很多，同时有些JIT编译器优化方式，比如分支预测，如果不进行profilling，往往并不能进行有效优化。


### 哪些是发生在编译时，运行时，或者两者都有？

方法重载发生在编译时，方法重载也被称为编译时多态，因为编译器根据参数的类型来选择使用哪个方法。
方法覆盖发生在运行时，方法覆盖被称为运行时多态，因为在编译期编译器不知道并且没法知道该去调用哪个方法。JVM会在代码运行的时候做出决定。

Java编译器会在编译时进行常量折叠，由于`final`变量的值不会改变，因此可以对它们进行优化。

泛型（又称类型检验）发生在编译期的。编译器负责检查程序中类型的正确性，然后把使用了泛型的代码翻译或者重写成可以执行在当前JVM上的非泛型代码。这个技术被称为“类型擦除“。换句话来说，编译器会擦除所有在尖括号里的类型信息，来保证和版本1。4。0或者更早版本的JRE的兼容性。

在运行时或者编译时都可以使用注解。
@Override可以用来捕获类似于在子类中把toString()写成tostring()这样的错误。在Java 5中，用户自定义的注解可以用注解处理工具(Anotation Process Tool ——APT)在编译时进行处理。到了Java 6，这个功能已经是编译器的一部分了。

@Test是JUnit框架用来在运行时通过反射来决定调用测试类的哪个（些）方法的注解。@Test (timeout=100)如果运行时间超过100ms的话，测试用例就会失败。

在运行时或者编译时都可以使用异常。

运行时异常（RuntimeException）也称作未检测的异常（unchecked exception），这表示这种异常不需要编译器来检测。RuntimeException是所有可以在运行时抛出的异常的父类。一个方法除要捕获异常外，如果它执行的时候可能会抛出RuntimeException的子类，那么它就不需要用throw语句来声明抛出的异常。

受检查异常（checked exception）都是编译器在编译时进行校验的，通过throws语句或者try{}cathch{} 语句块来处理检测异常。编译器会分析哪些异常会在执行一个方法或者构造函数的时候抛出。

面向切面的编程（Aspect Oriented Programming-AOP）：切面可以在编译时，运行时或，加载时或者运行时织入。

1. 编译期：编译期织入是最简单的方式。如果你拥有应用的代码，你可以使用AOP编译器（例如，ajc – AspectJ编译器）对源码进行编译，然后输出织入完成的class文件。AOP编译的过程包含了waver的调用。切面的形式可以是源码的形式也可以是二进制的形式。如果切面需要针对受影响的类进行编译，那么你就需要在编译期织入了。

2. 编译后：这种方式有时候也被称为二进制织入，它被用来织入已有的class文件和jar文件。和编译时织入方式相同，用来织入的切面可以是源码也可以是二进制的形式，并且它们自己也可以被织入切面。

3. 装载期：这种织入是一种二进制织入，它被延迟到JVM加载class文件和定义类的时候。为了支持这种织入方式，需要显式地由运行时环境或者通过一种“织入代理（weaving agent）“来提供一个或者多个“织入类加载器（weaving class loader）”。

4. 运行时：对已经加载到JVM里的类进行织入。

继承发生在编译时，因为它是静态的。

代理或者组合发生在运行时，因为它更加具有动态性和灵活性。

这也是为什么有`组合优于继承`这种说法，继承是一种多态工具，而不是一种代码复用工具。过度使用继承（通过“extends”关键字）的话，如果修改了父类，会损坏所有的子类。这是因为子类和父类的紧耦合关系是在编译期产生的。果你的类之间没有继承关系，并且你想要实现多态，那么你可以通过接口和组合的方式来实现，这样不仅可以实现代码重用，同时也可以实现运行时的灵活性。


### AOT

除了日常最常见的java使用模式，还有一种新的编译方式，即所谓的AOT(Ahead-of-Time Compilation)，直接将字节码编译成机器代码，这样就避免了JIT预热等各方面的开销，比如Oracle JDK 9 就引入了实验性的AOT特性，并且增加了新的jaotc工具，利用下面的命令吧某一类或者摸个模块编译成为AOT库。

```java

jaotc --output libHelloWorld.so HelloWorld.class
jaotc --output libjava.base.so --module java.base

```

然后，在启动时直接指定的指定就可以了。


```

java -XX:AOTLibrary=./libHelloWorld.so ./libjava.base.so HelloWorld

```




而且，Oracle JDK支持分层编译和AOT协作使用，并不是二选一的关系。AOT也不仅仅是只有这一种方式，业界早有第三方工具(如GCJ，Excelsior JET)提供相关功能。

另外，JVM作为一个强大的平台，不仅仅只有java语言可以运行在JVM上，本质上合规的字节码都可以运行，Java语言自身也为此提供了便利，类似Clojure、Scala、Groovy、Jruby、Jython等大量JVM语言，活跃在不同的场景。



### 提问

1. JVM的内存模型，堆、栈、方法区；字节码的跨平台性；对象在JVM中的强引用，弱引用，软引用，虚引用，是否可用finalise方法救救它？;双亲委派进行类加载，什么是双亲呢？双亲就是多亲，一份文档由我加载，然后你也加载，这份文档在JVM中是一样的吗？；多态思想是Java需要最核心的概念，也是面向对象的行为的一个最好诠释；理解方法重载与重写在内存中的执行流程，怎么定位到这个具体方法的。
2. 发展流程，JDK5(重写bug)，JDK6(商用最稳定版)，JDK7(switch的字符串支持)，JDK8(函数式编程)，一直在发展进化。
3. 理解祖先类Object，它的行为是怎样与现实生活连接起来的。
4. 理解23种设计模式，因为它是道与术的结合体。


-------------------
参考:

[知乎专栏](https://zhuanlan。zhihu。com/p/22886648)
[Java核心36讲](https://time。geekbang。org/column/article/6845)
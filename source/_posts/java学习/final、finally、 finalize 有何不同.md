---
title: final、finally、 finalize 有何不同
date: 2018-11-12 17:35:00
tags: 
- java
categories: 总结
---

其实这三者风马牛不相及,尽管这个问题很基础,但是除了语法和使用实践角度出发,还可以从其性能、并发、对象生命周期或垃圾收集基本过程等方面的理解.

<!-- more -->
### final

`final` 可以用来修饰类、方法、变量,修饰class代表不可以继承扩展,修饰变量代表不可以修改,修饰方法代表不可以重写.初衷就是明确告知别人这些行为是不许修改的.在一些第三方类库中的一些基础类会声明成final class,可以有效避免API使用者更改基础功能,在某种程度上,是保证平台安全的必要手段.

但是需要注意的是,`final`不是`immutable`! 不可变最基本是行为不可变，不提供可变的操作。变量私有化，没有增加，修改等方法，final只是保证指针不可变，无法保证内容不可变。

```java
 final List<String> strList = new ArrayList<>();
 strList.add("Hello");
 strList.add("world"); 
 //JDK 9 新特性,不可变
 List<String> unmodifiableStrList = List.of("hello", "world");
 unmodifiableStrList.add("again");
 //抛出异常
```

`final`只能约束`strList`这个引用不可被赋值,但是strList对象行为不被影响.

如果我们真的希望对象本身是不可变的,那么需要相应的类支持不可变的行为.目前Java语言没有原生的不可变支持,如果要实现`immutable`的类,我们需要做到:

* 将class自身声明为final,这样就不能通过扩展来绕过限制
* 将所有成员变量定义为private和final,并不要实现setter方法
* 构造对象时,成员变量使用深度拷贝来初始化,而不是直接赋值,防止被人对输入对象修改
* 如果要实现getter方法或者可能会返回内部状态的放,使用COW(copy-on-write)原则,创建私有的copy.

### finally

`finally `保证重点代码一定要被执行的机制.可以用 try-finally 或者 try-catch-finally 来进行类似关闭JDBC连接,保证unlock锁等动作.

```java

try {
  // do something
  System.exit(1);
} finally{
  System.out.println(“Print from finally”);
}

```
上面 finally 里面的代码可不会被执行的哦，这是一个特例.

### finalize

`finalize` 是基础类Object的一个方法,他的设计目的是保证对象在被垃圾收集前完成特定资源的回收.finalize 机制现在已经不推荐使用,并且在`JDK 9`开始被标记为`deprecated`.为什么呢?因为你无法保证 `finalize` 何时被执行,执行是否符合预期.使用不当会影响性能,导致程序死锁、挂起等.

通常来说,利用`try-with-resource` 或者`try-finally`机制来进行资源回收.如果需要额外处理们可以考虑Java提供的Cleaner机制或者其他替代方法.

一旦实现了非空的`finalize`方法,就会导致相应对象回收呈现数量级上的变慢,有人专门做过benchmark,大概是40~50倍的下降.

`finalize`被设计为在对象被垃圾收集前调用,finalize()方法是GC(garbage collector) 运行机制的一部分,finalize()方法的作用是什么呢？finalize()方法是在GC清理它所从属的对象时被调用的，如果执行它的过程中抛出了无法捕获的异常（uncaughtexception，GC将终止对改对象的清理，并且该异常会被忽略；直到下一次GC开始清理这个对象时，它的finalize()会被再次调用。这就意味着实现了`finalize`方法的对象,JVM要对他额外处理,本质上对快速回收进行了阻碍,可能导致
你的对象经过多个垃圾收集周期才能被回收.

Java平台目前逐步使用`java.lang.ref.Cleaner`来替换掉原有的`finalize`实现.Cleaner利用了幻象引用,这是一种常见的post-mortem(验尸) 清理机制,个Cleaner 的操作都是独立的，有自己的运行线程，避免意外死锁的问题。

注意，从可预测性的角度来判断，Cleaner 或者幻象引用改善的程度仍然是有限的，如果由于种种原因导致**幻象引用堆积**，同样会出现问题。所以，Cleaner 适合作为一种最后的保证手段，而不是完全依赖 Cleaner 进行资源回收，不然我们就要再做一遍 finalize 的噩梦了。

很多第三方库自己直接利用幻象引用定制资源收集，比如广泛使用的 MySQL JDBC driver 之一的 mysql-connector-j，就利用了幻象引用机制。幻象引用也可以进行类似链条式依赖关系的动作，比如，进行总量控制的场景，保证只有连接被关闭，相应资源被回收，连接池才能创建新的连接。

另外，这种代码如果稍有不慎添加了对资源的强引用关系，就会导致循环引用关系，前面提到的 MySQL JDBC 就在特定模式下有这种问题，导致内存泄漏。上面的示例代码中，将 State 定义为 static，就是为了避免普通的内部类隐含着对外部对象的强引用，因为那样会使外部对象无法进入幻象可达的状态。
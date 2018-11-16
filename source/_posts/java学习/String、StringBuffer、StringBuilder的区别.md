---
title: String、StringBuffer、StringBuilder的区别
date: 2018-11-16 23:03:00
tags: 
- java
categories: 总结
---

字符串在很多地方都是一个特殊的存在,今天来聊一聊他们之间的区别。

<!-- more -->

### String

String是Java语言非常基础和重要的类，提供了构造和管理字符串的各种基本逻辑。他是典型的Immutable类，被声明为 final class，所有属性都是 final 的。 也由于他的不可变性，类似拼接、裁剪字符串等动作，都会产生新的String对象。由于字符串操作的普遍性，所以相关操作的效率往往对应用性能有明显影响。这种便利体现在拷贝构造函数中，由于不可变，Immutable 对象在拷贝时不需要额外复制数据。

### StringBuffer

StringBuffer 是为解决上面提到的拼接产生太多中间对象额度问题而提供的类。我们可以用append或者add方法，把字符串添加到已有序列的末尾或者指定位置。StringBuffer本质是一个线程安全的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销，所以除非有线程安全的需要，不然还是推荐使用后继者，也就是 StringBuilder。他的线程安全是通过synchronized
关键字实现的，非常直白。这种实现方式非常适合我们常见的线程安全类实现，不必纠结于synchronized性能之类的。


### StringBuilder

StringBuilder 是Java 1.5新增的，在能力上和StringBuffer没有本质区别，但是它去掉了线程安全的部分，有效减少了开销，是绝大部分情况下进行字符串拼接的首选。

StringBuilder 和 StringBuffer 底层都是利用可修改的（char,JDK 9 以后是byte）数组,都继承了 `AbstractStringBuilder`, 里面包含了基本操作,区别仅在于最终的方法是否加了`synchronized`。

另外，这个内部数组应该创建多大呢？目前的实现是，构建时初始字符串长度加16，我们确定拼接会发生多次，而且大概是可预期的，那么可以指定合适的大小，避免多次扩容的开销，扩容会产生多重开销，因为要抛弃原有数组，创建新的数组，还要进行Arraycopy。

但其实，Java是相当智能的。

```java

  public class StringConcat {
        public static void main(String[] args) {
            String myStr = "aa" + "bb" + "cc" + "dd";   
             System.out.println("My String:" + myStr);   
        } 
    }

```

先编译再反编译,在JDK 8中，字符串拼接操作会自动被javac转换为StringBuilder操作，而在JDK 9 里面,为了更加统一字符串操作优化，提供了`StringConcatFactory`，作为一个统一的入口。

### 字符串缓存

把常见应用进行堆转储(Dump Heap)，然后分析对象组成，会发现平均25%的对象是字符串，其中约半数是重复的。如果能避免创建重复字符串，可以有效降低内存消耗和对象创建的开销。

String 在Java 6 以后提供了intern() 方法，目的是提示JVM把相应字符串缓存起来，以备重复使用。但是在Java 6中，不推荐使用 intern() ，因为被缓存的字符串是存在所谓`PermGen`里的，也就是臭名昭著的“永久代”，这个空间是很有有限的，也基本不会被FullGc之外的垃圾收集照顾到。所以，使用不当，OOM就会光顾。

在后续的版本中，这个缓存被放在了堆中，甚至永久代在JDK 8 中被 MetaSpace（元数据区）替代了。而且，默认缓存大小也在不断扩大中，从最初的1009，到7u40以后被修改为60013。可以使用下面参数直接打印具体数字,也可以设置。

```
-XX:+PrintStringTableStatistics
```

Intern 是一种显式地排重机制，但是它也有一定的副作用，因为需要开发者写代码时明确调用。另外很难保证效率，应用开发阶段很难清楚地预计字符串的重复情况，有人认为是一种污染代码的实践。

在 Oracle JDK 8u20 之后，推出了一个新的特性，也就是G1 GC下的字符串排重。它是通过将相同数据的字符串指向同一份数据来做到的，是JVM底层的改变，不需要Java类库的改动。另，这个功能时默认关闭的。

在运行时，字符串的一些基础操作会利用JVM内部的Intrinsic机制，往往运行的就是特殊优化的本地代码，而根本就不是Java代码生成的字节码。Intrinsic可以简单理解为，是一种利用native方式hard-coded的逻辑,算是一种特别的内连。

### String自身的演化

如果你仔细观察过Java的字符串，在历史版本中，他是使用char 数组来存数据的,这样非常直接。但是Java中的 char 是两个 byte 大小,拉丁语系的字符,根本不需要太宽的char,造成一定的浪费。因此归根结底绝大部分任务是用来操作数据的。

其实在Java 6中，Oracle JDK 就提供了压缩字符串的特性，但是这个特性在最新的JDK中已经被移除了。

在 Java 9 中,引入了Compact Strings 的设计,对字符串进行了大刀阔斧的改进，将数据存储方式从char数组，改变为一个byte数组加上一个标识编码的所谓coder，并且将相关字符串操作类都进行了修改。另外，所有相关的Intrinsic之类都进行了重写，以保证没有任何性能损失。




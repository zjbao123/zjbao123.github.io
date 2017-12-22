---
title: 初探python(3)高级特性
date: 2016-09-20 20:14:50
tags: 
- python
- python教程
categories: 总结
---


在Python中，并不是代码越多越好，而是越少越好。基于这个思想，我们来介绍一下Python的高级特性来精简我们的代码。

## 切片
取一个list或tuple的部分元素是非常常见的操作。对这种经常取指定索引范围的操作，用循环十分繁琐，因此，Python提供了切片（Slice）操作符，能大大简化这种操作。
<!-- more -->
```
L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']

```

```
L[0:3]
L[:3]
L[1:3]
 L[-2:]
```
若要取前三个元素，不需要用循环.直接一行代码搞定。第一行
表示从索引0开始取，直到索引3为止，但不包括索引3。也可以取负数，则表示从尾部开始取。

如果从0开始取，0还可以省略。

记住倒数第一个元素的索引是-1。
```
L = range(100)#0~99数列
L[:10:2]#从第一个开始，0到10每两个取一个，就跳一个取
L[::5]#所有数每5个取一个
L[:]#复制一遍
'ABCDEFG'[:3]#'ABC'
'ABCDEFG'[::2]#'ACEG'
```
tuple也可以切，结果也是一个tuple

Python没有针对字符串的截取函数，只需要切片一个操作就可以完成，非常简单。

## 迭代
如果给定一个list或tuple，我们可以通过for循环来遍历这个list或tuple，这种遍历我们称为迭代（Iteration）。

任何可迭代对象都可以作用于for循环，包括我们自定义的数据类型，只要符合迭代条件，就可以使用for循环。
```
d = {'a': 1, 'b': 2, 'c': 3}
for key in d:
    print key
#a b c
```
由于字符串也是可迭代对象，因此，也可以作用于for循环。

默认情况下，dict迭代的是key。如果要迭代value，可以用`for value in d.itervalues()`，如果要同时迭代key和value，可以用`for k, v in d.iteritems()`。

那么，如何判断一个对象是可迭代对象呢？方法是通过collections模块的Iterable类型判断。
```
from collections import Iterable
isinstance('abc', Iterable)
```

如果要对list实现类似Java那样的下标循环怎么办？Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身：
```
for i, value in enumerate(['A', 'B', 'C']):
    print i, value
#0 A
#1 B
#2 C
```

## 列表生成式
列表生成式即List Comprehensions，是Python内置的非常简单却强大的可以用来创建list的生成式。

```
[x * x for x in range(1, 11)]
[m + n for m in 'ABC' for n in 'XYZ']

```
但是循环太繁琐，而列表生成式则可以用一行语句代替循环生成所需的list。
也可以用双循环来生成全排列。

运用列表生成式，可以写出非常简洁的代码。例如，列出当前目录下的所有文件和目录名，可以通过一行代码实现：
```
import os # 导入os模块，模块的概念后面讲到
[d for d in os.listdir('.')] # os.listdir可以列出文件和目录
```

运用列表生成式，可以快速生成list，可以通过一个list推导出另一个list，而代码却十分简洁。

如果list中既包含字符串，又包含整数，由于非字符串类型没有lower()方法.
```
L = ['Hello','world',2016,0116]

print [s.lower() if isinstance(s, str) else s for s in L]
```
## 生成器

通例如创建一个包含100万个元素的列表，且不说由于内存限制，不仅占用很大的存储空间，而且我们通常仅仅需要访问前面几个元素，因此，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器（Generator）。

要创建一个generator，只需把列表生成式中的`[]`改成`（）`就行了，也就是元组。

`generator`列出所有元素需要通过`generator`的`next()`方法，一个一个打印出来。

generator保存的是算法，每次调用next()，就计算出下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration的错误。

当然，上面这种不断调用next()方法实在是太变态了，正确的方法是使用for循环，因为generator也是可迭代对象.

所以基本上不会用`next()`方法，一般都用for循环。

generator非常强大。如果推算的算法比较复杂，用类似列表生成式的for循环无法实现的时候，还可以用函数来实现。

例如，打印斐波拉契数列，用列表上生成式很难写，不过generator就很方便
```
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print b
        a, b = b, a + b
        n = n + 1
```

fib函数实际上是定义了斐波拉契数列的推算规则，可以从第一个元素开始，推算出后续任意的元素，这种逻辑其实非常类似generator。

上面的函数和generator仅一步之遥。要把fib函数变成generator，只需要把print b改为yield b就可以了：
```
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
```
这就是定义generator的另一种方法。如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator。
```
for n in fib(6):
    print n
#######
1
1
2
3
5
8
```
### 生成器工作流程

这里最难理解的就是generator和函数的执行流程不一样。

函数是顺序执行，遇到return语句或者最后一行函数语句就返回。

而变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。


它的工作流程是在for循环的过程中不断计算出下一个元素，并在适当的条件结束for循环。对于函数改成的generator来说，遇到return语句或者执行到函数体最后一行语句，就是结束generator的指令，for循环随之结束。




>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。


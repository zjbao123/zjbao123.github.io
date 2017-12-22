---
title: 初探python(4)函数式编程
date: 2016-09-26 20:14:50
tags: 
- python
- python教程
categories: 总结
---


我们首先要搞明白计算机（Computer）和计算（Compute）的概念。

在计算机的层次上，CPU执行的是加减乘除的指令代码，以及各种条件判断和跳转指令，所以，汇编语言是最贴近计算机的语言。

而计算则指数学意义上的计算，越是抽象的计算，离计算机硬件越远。
<!-- more -->
函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，Python不是纯函数式编程语言。

## 高阶函数(Higher-order function）

### 变量可以指向函数
```
f = abs
f
<built-in function abs>
```
函数本身也可以赋值给变量，即：变量可以指向函数。

### 函数名也是变量

对于`abs()`这个函数，完全可以把函数名abs看成变量，它指向一个可以计算绝对值的函数。那如果把abs指向其他对象，就无法通过`abs(-10)`调用该函数了！

这里是为了说明函数名也是变量，实际代码绝对不能这么写。如要恢复abs函数，请重启Python交互环境。

注：由于abs函数实际上是定义在`__builtin__`模块中的，所以要让修改abs变量的指向在其它模块也生效，要用`__builtin__.abs = 10`。

### 传入函数
既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

```
def add(x, y, f):
    return f(x) + f(y)
```
当我们调用add(-5, 6, abs)时，参数x，y和f分别接收-5，6和abs，根据函数定义，计算结果就为11。

### map/reduce
如果你读过Google的那篇大名鼎鼎的论文:[MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html),你就能大概明白map/reduce的概念。
Python内建了`map()`和`reduce()`函数。

#### map

map()函数接收两个参数，一个是函数，一个是序列，map将传入的函数依次作用到序列的每个元素，并把结果作为新的list返回。

#### reduce
reduce把一个函数作用在一个序列[x1, x2, x3...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算。原理就是：
```
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```
#### 练习
1.利用map()函数，把用户输入的不规范的英文名字，变为首字母大写，其他小写的规范名字。输入：['adam', 'LISA', 'barT']，输出：['Adam', 'Lisa', 'Bart']。
写法一：
```
def f(x):
    return x.capitalize()
print map(f,['adam', 'LISA', 'barT'])
```
写法二：
```
def f(x):
    return x[0].upper() + x[1:].lower()
map(f,['adam', 'LISA', 'barT'])

```
2.Python提供的sum()函数可以接受一个list并求和，请编写一个prod()函数，可以接受一个list并利用reduce()求积。
```
def f(x,y):
    return x*y
L=range(1,5)
print reduce(f,L)
```

### filter

正如名字所说的那样，Python内建的`filter()`函数用于过滤序列。

`filter()`也也接收一个函数和一个序列。filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

#### 练习

请尝试用filter()删除1~100的素数。

解：首先，1不是素数。其次，一个数m，如果m不能被2~√m间任一整数整除，m必定是素数。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'
import math
L=range(101)
def f(x):
    if x==1:
        return 1
    else:
        for i in range(2,int(math.sqrt(x))+1):
            if x%i==0:
                return 1

print (filter(f,L))
```
### sorted
Python内置的sorted()函数就可以对list进行排序。

此外，sorted()函数也是一个高阶函数，它还可以接收一个比较函数来实现自定义的排序。比较的过程必须通过函数抽象出来。通常规定，对于两个元素x和y，如果认为x < y，则返回-1，如果认为x == y，则返回0，如果认为x > y，则返回1，这样，排序算法就不用关心具体的比较过程，而是根据比较结果直接排序。

比如，如果要倒序排序，我们就可以自定义一个reversed_cmp函数:
```
def reversed_cmp(x, y):
    if x > y:
        return -1
    if x < y:
        return 1
    return 0
```
传入自定义的比较函数reversed_cmp，就可以实现倒序排序:
```
sorted([36, 5, 12, 9, 21], reversed_cmp)
#[36, 21, 12, 9, 5]
```
注意区别sort(),是可变对象(字典、列表)的方法，无参数，无返回值，sort()会改变可变对象，因此无需返回值。sort()方法是可变对象独有的方法或者属性，而作为不可变对象如元组、字符串是不具有这些方法的，如果调用将会返回一个异常。

### zip()
```
>>> help(zip)
Help on built-in function zip in module __builtin__:
 
zip(...)
     zip(seq1 [, seq2 [...]]) -> [(seq1[0], seq2[0] ...), (...)]
     
     Return a list of tuples, where each tuple contains the i-th element
     from each of the argument sequences.  The returned list is truncated
     in length to the length of the shortest argument sequence.

```
zip([seql, ...])接受一系列可迭代对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。若传入参数的长度不等，则返回列表的长度和参数中长度最短的对象相同。

```
name = {"bob", 'alice', 'hello'}
age = {12, 3, 45}
for n in zip(name, age):
    print n

#('bob', 3)
#('hello', 12)
#('alice', 45)
```
zip()配合*号操作符,可以将已经zip过的列表对象解压。
```
n=zip(name, age)
print zip(*n)
#[('bob', 'hello', 'alice'), (3, 12, 45)]
```
利用zip()配合*这个特性，我们可以用来反转矩阵。
```
a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print zip(*a)

#[(1, 4, 7), (2, 5, 8), (3, 6, 9)]
```
使用zip合并相邻的列表项,两个iter(a)指向同一对象,因此每对zip都是对同一对象迭代两次。
```
a = [1, 2, 3, 4, 5, 6]
print zip(*([iter(a)] * 2))
#[(1, 2), (3, 4), (5, 6)]
```
### enumerate
先来看看官方说明：
```
Return an enumerate object.  iterable must be another object that supports
iteration.  The enumerate object yields pairs containing a count (from
start, which defaults to zero) and a value yielded by the iterable argument.
enumerate is useful for obtaining an indexed list:
    (0, seq[0]), (1, seq[1]), (2, seq[2]), ...
```
返回一个枚举类型，可迭代的对象，enumerate将其组成一个索引序列，利用它可以同时获得索引和值。通常用于计数。

在统计文件行数时，按照老方法写，文件大的时候可能会停止工作：
```
print len(open("seq.txt").readlines())
```
有了枚举，可以这样写：
```
count = -1 
for index, line in enumerate(open(filepath,'r'))： 
    count += 1
```
### reversed()
```
reverse iterator over values of the sequence
```
反转函数,无论传递什么参数，都将返回一个以列表为容器的返回值，如果是字典将返回键的列表。

注意区别revers(),是可变对象(字典、列表)的方法，无参数，无返回值，sort()会改变可变对象，因此无需返回值。sort()方法是可变对象独有的方法或者属性，而作为不可变对象如元组、字符串是不具有这些方法的，如果调用将会返回一个异常。
```
mylist=[5,4,3,2,1]
mylist.reverse()
print mylist
```
##[::-1]
通过序列的切片也可以达到“逆转”的效果。这也是一种`slice`的特殊写法。
## 返回函数
高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。

我们来实现一个可变参数的求和。通常情况下，求和的函数是这样定义的：
```
def calc_sum(*args):#有序列表
    ax = 0
    for n in args:
        ax = ax + n
    return ax
```
如果不需要立刻求和，而是在后面的代码中，根据需要再计算怎么办？可以不返回求和的结果，而是返回求和的函数！
```
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum
```

```
f = lazy_sum(1, 3, 5, 7, 9)
>>> f
<function sum at 0x10452f668>
>>> f()
25

```

只有真的调用这个函数，才会真正计算出结果。

当lazy_sum返回函数sum时，相关参数和变量都保存在返回的函数中，这种称为“闭包（Closure）”的程序结构拥有极大的威力。
```
f1=lazy_sum(1, 3, 5, 7, 9)
f2=lazy_sum(1, 3, 5, 7, 9)
```
当我们调用lazy_sum()时，每次调用都会返回一个新的函数，即使传入相同的参数，f1()和f2()的调用结果互不影响。

### 闭包
要注意的是，返回的函数并没有立刻执行，而是直到调用了f()才执行。
```
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()

>>> f1()
9
>>> f2()
9
>>> f3()
9

```
原因就在于返回的函数引用了变量i，但它并非立刻执行。等到3个函数都返回时，它们所引用的变量i已经变成了3，因此最终结果为9。

返回闭包时牢记的一点就是：**返回函数不要引用任何循环变量，或者后续会发生变化的变量。**

如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变。
```
def count():
     fs = []
    for i in range(1, 4):
        def f(j):
            def g():
                return j*j
            return g
        fs.append(f(i))
    return fs
 
>>> f1, f2, f3 = count()
>>> f1()
1
>>> f2()
4
>>> f3()
9

```
最重要的是要理解f1, f2, f3 = count()这句话。

count()的返回值是一个list，list里面存的是执行for i in range(1, 4)后返回的3个fs函数
f1, f2, f3 = count()将这3个fs函数依次赋值给f1, f2, f3；同时赋值的还有变量j。因为j=i让j和当前的i绑定起来了，所以f1, f2, f3得到的j值分别是1,2,3

最后在计算f1()的时候，用的是之前传过来的j=1，而不是变化后的i=3，所以f1()=1。 f2(),f3(),依次类推。

## 匿名函数
```
 map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9])
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```
关键字lambda表示匿名函数，冒号前面的x表示函数参数。

匿名函数有个限制，就是只能有一个表达式，不用写return，返回值就是该表达式的结果。

用匿名函数有个好处，因为函数没有名字，不必担心函数名冲突。此外，匿名函数也是一个函数对象，也可以把匿名函数赋值给一个变量，再利用变量来调用该函数。

同样，也可以把匿名函数作为返回值返回,
```
def build(x, y):
    return lambda a,b: x * a + y * b

print build(1,2)(3,4)

```

Python对匿名函数的支持有限，只有一些简单的情况下可以使用匿名函数。

## 装饰器

假设我们要增强函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改函数的定义，这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

装饰器实则是一个返回函数的高阶函数。假如，我们要定义一个能打印日志的decorator，可以定义如下：
```
def log(func):
    def wrapper(*args, **kw):
        print 'call %s():' % func.__name__
        return func(*args, **kw)
    return wrapper

```
观察上面的log，因为它是一个decorator，所以接受一个函数作为参数，并返回一个函数。我们要借助Python的@语法，把decorator置于函数的定义处：
```
@log
def now():
    print '2013-12-25'
```
把@log放到now()函数的定义处，相当于执行了语句：
```
now = log(now)

```
wrapper()函数的参数定义是(*args, **kw)，因此，wrapper()函数可以接受任意参数的调用。在wrapper()函数内，首先打印日志，再紧接着调用原始函数。

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：
```
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print '%s %s():' % (text, func.__name__)
            return func(*args, **kw)
        return wrapper
    return decorator

@log('execute')
def now():
    print '2016-09-25'


>>> now()
execute now():
2016-09-25

```
3层嵌套的效果是这样的:
```
now = log('execute')(now)
```
我们来剖析上面的语句，首先执行log('execute')，返回的是decorator函数，再调用返回的函数，参数是now函数，返回值最终是wrapper函数。

以上两种decorator的定义都没有问题，但还差最后一步。因为我们讲了函数也是对象，它有__name__等属性，但你去看经过decorator装饰之后的函数，它们的__name__已经从原来的'now'变成了'wrapper'。

因为返回的那个wrapper()函数名字就是'wrapper'，所以，需要把原始函数的__name__等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。

因此，可以通过Python内置的functools.wraps来它保留原有函数的名称和docstring。


### 练习

1.请编写一个decorator，能在函数调用的前后打印出'begin call'和'end call'的日志。

解答如下：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'
import functools
def decorator(func):
    @functools.wraps(func)
    def wrapper(*args,**kwargs):
        print "begin call %s" % func.__name__
        m=func(*args,**kwargs)
        print "end call %s" % func.__name__
        return m
    return wrapper

@decorator
def new():

    print "2016-09-25"

new()
```

2.再思考一下能否写出一个@log的decorator，使它既支持：
```
@log
def f():
    pass
```
又支持：
```
@log('execute')
def f():
    pass
```

解：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'
import functools
def log(txt="hello"):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args,**kwargs):
            print "%s begin call %s" % (txt,func.__name__)
            m=func(*args,**kwargs)
            print "%s end call %s" % (txt,func.__name__)
            return m
        return wrapper
    return decorator
@log('execute')
def f():
   pass

f()

@log()
def f():
    pass

f()

```

## 偏函数
上面提到了`·functools`模块，它有很多功能，其中之一就是偏函数。要注意，这里的偏函数和数学意义上的偏函数不一样。

`functools.partial`就是帮助我们创建一个偏函数的，不需要我们自己定义`int2()`，可以直接使用下面的代码创建一个新的函数`int2`：
```
import functools
int2 = functools.partial(int, base=2)
```
当函数的参数个数太多，需要简化时，使用`functools.partial`可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。

调用方法就是：`functools.partial(函数名，要固定的参数名并赋值)`

所以先要通过`help()`函数来找到参数名，再进行固定。

然而，这个函数仅仅是把默认值进行了修改，人为调用的时候，还是可以赋值。



>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)和官方文档的一个学习内容的总结。以便于自己后的学习和整理。
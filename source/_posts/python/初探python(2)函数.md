---
title: 初探python(2)函数
date: 2016-09-19 22:14:50
tags: 
- python
- python教程
categories: 总结
---

Python内置了很多有用的函数，我们可以直接调用。

如果要在函数内调用全局函数，应使用`global`。

## 函数定义
在Python中，函数的定义要用`def`，下面给一个示例。
```
def my_abs(x):
    if x >= 0:
        return x
    else:
        return -x
```
<!-- more -->
如果没有`return`语句，函数执行完毕后也会返回结果，只是结果为`None`。
`return None`可以简写为`return`。

## 空函数
想定义一个空函数，可以用`pass`语句。`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

## 参数检查
调用函数时，如果参数个数不对，`Python`解释器会自动检查出来，并抛出`TypeError`。
而参数类型的错误则无法检查，应该在定义函数时完善。让我们修改一下`my_abs`的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数`isinstance`实现(isinstance用来判断变量类型)：
```
def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```
##　返回多个值
python可以返回多个值是以元组(tuple)来实现的。返回一个tuple可以省略括号，而多个变量可以同时接收一个tuple，按位置赋给对应的值。

## 默认参数
默认参数用法都与其他语言保持一致。用法如下：
1.必选参数在前，默认参数在后，否则Python的解释器会报错
2.二是如何设置默认参数。

当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数。
使用默认参数有什么好处？最大的好处是能降低调用函数的难度。

定义默认参数要牢记一点：默认参数必须指向不变对象！
```
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```
可以用上述说的None这个不可变对象来实现。

为什么要设计str、None这样的不变对象呢？因为不变对象一旦创建，对象内部的数据就不能修改，这样就减少了由于修改数据导致的错误。此外，由于对象不变，多任务环境下同时读取对象不需要加锁，同时读一点问题都没有。我们在编写程序时，如果可以设计一个不变对象，那就尽量设计成不变对象。

## 可变参数

如果要定义不定数量的参数的话，我们首先想到可以把a，b，c……作为一个`list`或`tuple`传进来。也可以使用可变参数。
```
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
定义可变参数和定义`list`或`tuple`参数相比，仅仅在参数前面加了一个*号。在函数内部，参数numbers接收到的是一个tuple，因此，函数代码完全不变。但是，调用该函数时，可以传入任意个参数，包括0个参数。

如果要传入一个已经写好的`tuple`的话，Python允许你在`list`或`tuple`前面加一个*号，把`list`或`tuple`的元素变成可变参数传进去：
```
nums = [1, 2, 3]
calc(*nums)
//14
```

## 关键字参数

关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个`dict`。
```
def person(name, age, **kw):
    print 'name:', name, 'age:', age, 'other:', kw
```

在调用该函数时，可以只传入必选参数,不填关键字参数
```
person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```
也可传入任意值。

关键字参数既可以直接传入：func(a=1, b=2)，又可以先组装dict，再通过*\*kw传入：func(**{'a': 1, 'b': 2})。

## 递归函数
递归的使用方法与其他相同，即自身调用自身。
```
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
```

**使用递归函数需要注意防止栈溢出。**

函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。

解决递归调用栈溢出的方法是通过尾递归优化，事实上尾递归和循环的效果是一样的，所以，把循环看成是一种特殊的尾递归函数也是可以的。

尾递归是指，在函数返回的时候，调用自身本身，并且，return语句不能包含表达式。这样，编译器或者解释器就可以把尾递归做优化，使递归本身无论调用多少次，都只占用一个栈帧，不会出现栈溢出的情况。

上面的`fact(n)`函数由于`return n * fact(n - 1)`引入了乘法表达式，所以就不是尾递归了。要改成尾递归方式，需要多一点代码，主要是要把每一步的乘积传入到递归函数中：
```
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)
```
可是，Python标准的解释器没有针对尾递归做优化，任何递归函数都存在栈溢出的问题。



>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
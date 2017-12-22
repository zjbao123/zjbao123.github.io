---
title: 初探python(6)面向对象编程
date: 2016-10-06 20:26:30
tags: 
- python
- python教程
categories: 总结
---


面向对象编程——Object Oriented Programming，简称OOP，是一种程序设计思想。OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。

由于这一章在C++中学的不错，所以内容较少。
<!-- more -->
## 面向对象与面向过程的区别
之前的面向过程的程序设计是把计算机程序视为一系列的命令集合，即一组函数的顺序执行。为了简化程序设计，面向过程把函数继续切分为子函数，即把大块函数通过切割成小块函数来降低系统的复杂度。

而面向对象的程序设计把计算机程序视为一组对象的集合，而每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间传递。

在Python中，所有数据类型都可以视为对象，当然也可以自定义对象。自定义的对象数据类型就是面向对象中的类（Class）的概念。

面向对象的设计思想更为自然，但其抽象程度要高于函数，另外，还有其三大特点：数据封装、继承和多态。

## 类和实例

类是抽象的模板，比如Student类，而实例是根据类创建出来的一个个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。

```
class student(object):
    def __init__(self, name, score):
        self.name = name
        self.score = score
    def print_imformation(self):
        print "your name is %s,your number is %s" % (self.name, self.number)

```
`class`后跟类名，`object`指的是继承的类，如果没有合适的类，就用object类，所有类都会最终继承它。
`__init__` 方法第一个参数是`self`，表示创建的实例本身，并且调用的时候，不用传递该参数。其他和普通函数没有什么区别。从而实现了对数据的封装。

好处其一：方便调用，不需要知道内部实现细节；
其二：可以给类增加新的方法。

## 访问限制

如果要让内部属性不被外部访问，可以把属性的名称前面加两个下划线，在`Python`中，这样就成了私有变量，只有内部可以访问。

如果希望在外部进行访问和修改，可以增加`get_name`和`set_name`这样的方法：
```
    def get_name(self):
        return self.__name

    def set_name(self):
        self.__name = name

```
需要注意的是，变量名是以双下划线开头，并且双下划线结尾的是特殊变量，特殊变量是可以直接访问的。

另外，变量名以一个下划线开头的，这样的实例变量其实可以外部访问，但按照约定俗称的规定，看到这样的变量时，应该讲他看做私有变量，不要随意访问。

双划线开头的实例变量不是一定不能访问，不能直接访问·`__name`是因为Python解释器对外把`__name`变量改成了`_Student__name`，所以，仍然可以通过`_Student__name`来访问`__name`变量。

但是强烈建议不能这么干，不同版本的Python解释器可能会把`__name`改为不同的变量名。

## 继承和多态

当我们定义一个class的时候，可以从某个现有的class继承，新的class称为子类（Subclass），而被继承的class称为基类、父类或超类（Base class、Super class）。

继承可以把父类的所有功能都直接拿过来，这样就不必重零做起，子类只需要新增自己特有的方法，也可以把父类不适合的方法覆盖重写；

有了继承，才能有多态。在调用类实例方法的时候，尽量把变量视作父类类型，这样，所有子类类型都可以正常被接收；

## 获取对象信息

当我们拿到一个对象的引用时，如道这个对象是什么类型？有什么方法呢？

### type()
基本类型都可以用type()函数来判断。
另外，有一种类型就叫`TypeType`，所有类型本身的类型就是`TypeType`。

```
type(123)
```
### isinstance()

对于class的继承关系，使用`type()`就很不方便。我们要判断`class`的类型，可以使用`isinistance()`函数。

### dir()
使用dir()函数可以获得一个对象的所有属性和方法。类似`__xxx__`的属性和方法在Python中都是有特殊用途的,比如`__len__`方法返回长度。在Python中，如果你调用`len()`函数试图获取一个对象的长度，实际上，在`len()`函数内部，它自动去调用该对象的`__len__()`方法。

因此，我们自己写的类，如果也想用`len(myObj)`的话，就自己写一个`__len__()`方法。

## 操作对象的状态

我们可以通过`getattr()`、`setattr()`以及`hasattr()`来操作对象的属性(attribute)。

```
#getattr() 得到属性值

#setattr() 设置属性值

#hasattr() 查看是否有该属性



class MyObject(object):
    def __init__(self):
        self.x = 9

    def power(self):
        return self.x * self.x

obj = MyObject()
print hasattr(obj, 'x')
print hasattr(obj, 'y')
setattr(obj, 'y', 19)
print hasattr(obj, 'y')
print getattr(obj, 'y'，404)  #如果值不存在返回值404
n = getattr(obj, 'power')
print n()

# 输出结果：true，false，true，19，81

```

>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
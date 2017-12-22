---
title: 初探python(7)面向对象高级编程
date: 2016-10-10 21:26:30
tags: 
- python
- python教程
categories: 总结
---

数据封装、继承和多态只是面向对象程序设计中最基础的3个概念。在Python中，面向对象还有很多高级特性，允许我们写出非常强大的功能。我们接下来会讨论多重继承、定制类、元类等高级概念。

<!-- more -->
## 引言

正常情况下，当我们定义了一个class，创建了一个class的实例后，我们可以给该实例绑定任何属性和方法，这就是动态语言的灵活性。
```
# 绑定属性
s = Student()
s.name = 'Michael'

# 绑定方法
from types import MethodType
s.set_age = MethodType(set_age, s, Student) # 给实例绑定一个方法

Student.set_score = MethodType(set_score, None, Student) # 给class绑定一个方法

```
动态绑定允许我们在程序运行的过程中动态给class加上功能，这在静态语言中很难实现。

## 使用`__slots__`

`__slots__`用以限制class的属性，Python允许在定义class的时候，定义一个特殊的`__slots__`变量，来限制该class能添加的属性。
```
 class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```
使用`__slots__`要注意，`__slots__`定义的属性仅对当前类起作用，对继承的子类是不起作用的。

如果要使得在子类中起作用，那要在子类中也定义`__slots__`，这样，子类允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。
```
class Student(object):
    __slots__ = ('age', 'name')
class new(Student):
    __slots__ = ('grade')

s = new()
s.name = 'a'
s.grade = 97
print s.grade
```

## 使用`@property`

在绑定属性的时候，如果我们把属性直接暴露出去，其值容易被随意更改。
Python内置的`@property`装饰器就是负责把一个方法变成属性调用的。

好处有二：
1.能检查参数
2.方便调用，可以用类似属性的方式来访问

```
class Student(object):
    @property
    def score(self):
        return self._score

    @score.setter
    def score(self,value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if 0<= value <= 100:
            self._score = value
        else:
            raise ValueError("取值范围为0到100！")
    @score.deleter
    def score(self):
        del self._score
        print "score is already delete"

s = Student()
s.score = 99
print s.score
del s.score

s = Student()
s.score = 99
print s.score
# 99
# score is already delete
```

我们在对实例属性操作的时候，就知道`@property`很可能不是直接暴露的，而是通过`getter`、`setter` 和`deleter`方法来实现的。

把一个`getter`方法变成属性，只需要加上`@property`就可以了，此时，`@property`本身又创建了另一个装饰器`@score.setter`，负责把一个`setter`方法变成属性赋值。

当然，你也有可能看到一些遗留的代码：
```
class Student(object):
    def get_score(self):
        return self._score
    def set_score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
    score = property(get_score, set_score)



s = Student()
s.score = 99
print s.score
```

通过增加一句`score = property(get_score, set_score)`，它允许score属性设置并获取值本身而不破坏原有代码。


## 多重继承

通过多重继承，一个子类就可以同时获得多个父类的所有功能。

### Mixin

如果不是单一继承，要实现多个继承，如让`Ostrich`除了继承自`Bird`外，再同时继承`Runnable`。这种设计通常称之为Mixin。

Mixin的目的就是给一个类增加多个功能，这样，在设计类的时候，我们优先考虑通过多重继承来组合多个Mixin的功能，而不是设计多层次的复杂的继承关系。

这样一来，我们不需要复杂而庞大的继承链，只要选择组合不同的类的功能，就可以快速构造出所需的子类。

注：Java没有此类设计。

## 定制类
Python的class中还有许多`__slots__`，`__len__()`这样有特殊用途的函数，可以帮助我们定制类。

### `__str__`和`__repr__`

用来定制打印实例的内容。
```
class Student(object):
    def __init__(self, name):
        self.name = name

print Student('Michael')
# <__main__.Student object at 0x109afb190>

# 在类中定义加一条
def __str__(self):
        return 'Student object (name: %s)' % self.name

print Student('Michael')
# Student object (name: Michael)
```
另外，发现直接这样写还是会结果不同
```
 s = Student('Michael')
 s
# <__main__.Student object at 0x109afb310>
```
因为直接显示变量调用的不是`__str__()`，而是`__repr__()`，两者的区别是`__str__()`返回用户看到的字符串，而`__repr__()`返回程序开发者看到的字符串，也就是说，`__repr__()`是为调试服务的。

因此，在定义里再加一条，省时省力的写法：
```
__repr__ = __str__
```

### `__iter__`
用于`for···in`循环，返回一个迭代对象。然后，Python的for循环就会不断调用该迭代对象的next()方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环。
```
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1  # 初始化两个计数器a，b

    def __iter__(self):
        return self  # 实例本身就是迭代对象，故返回自己

    def next(self):
        self.a, self.b = self.b, self.a + self.b  # 计算下一个值
        if self.a > 100000:  # 退出循环的条件
            raise StopIteration()
        return self.a  # 返回下一个值
for n in Fib():
    print n
```

### `__getitem__`
如上所说，Fib实例虽然能作用于for循环，看起来和list有点像，但是，把它当成list来使用还是不行，取不了其中的元素。
```
class Fib(object):
    def __getitem__(self, n):
        a, b = 1, 1
        for x in range(n):
            a, b = b, a + b
        return a
```
这样，通过调用`print Fib()[2016]`就可以访问第2016项了（第0项就是原来数）。

如果要实现如同`list`的切片操作，需判断传入值是slice还是int：
```
class Fib(object):
    def __getitem__(self, n):
        if isinstance(n, int):
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a
        if isinstance(n, slice):
            start = n.start
            stop = n.stop
            a, b = 1, 1
            L = []
            for x in range(stop):
                if x >= start:
                    L.append(a)
                a, b = b, a + b
            return L
```
但是上述例子中没有对step参数作处理，也没有对负数作处理，所以，要正确实现一个`__getitem__()`还是有很多工作要做的。

此外，如果把对象看成dict，__getitem__()的参数也可能是一个可以作key的object，例如str。

与之对应的是__setitem__()方法，把对象视作list或dict来对集合赋值。最后，还有一个__delitem__()方法，用于删除某个元素。

总之，通过上面的方法，我们自己定义的类表现得和Python自带的list、tuple、dict没什么区别，这完全归功于动态语言的“鸭子类型”（“鸭子类型”的语言是这么推断的：一只鸟走起来像鸭子、游起泳来像鸭子、叫起来也像鸭子，那它就可以被当做鸭子。也就是说，它不关注对象的类型，而是关注对象具有的行为(方法)。），不需要强制继承某个接口。


### `__getattr__`

正常情况下，当我们调用类的方法或属性时，如果不存在，就会报错。这个方法的优点就是动态返回一个属性。

当调用不存在的属性时，比如score，Python解释器会试图调用__getattr__(self, 'score')来尝试获得属性。
```
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr=='score':
            return 99
s.score
# 99
```
这种完全动态调用的特性有什么实际作用呢？作用就是，可以针对完全动态的情况作调用。

作用之一就是动态调用API，利用完全动态的`__getattr__`，我们可以写出一个链式调用：
```
class Chain(object):

    def __init__(self, path=''):
        self._path = path

    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))

    def __str__(self):
        return self._path
print Chain().status.user.timeline.list
# '/status/user/timeline/list'
#调用Github的URL时，需要把:user替换为实际用户名。

```
这样就能很方便的调用API了。

### `__call__`

`__call__`能够直接就可以对实例进行调用。

```
class Student(object):
    def __init__(self, name):
        self.name = name

    def __call__(self):
        print('My name is %s.' % self.name)
s = Student('Michael')
s()
# My name is Michael.
```

对实例进行直接调用就好比对一个函数进行调用一样，所以你完全可以把对象看成函数，把函数看成对象，因为这两者之间本来就没啥根本的区别。

那么，怎么判断一个变量是对象还是函数呢？其实，更多的时候，我们需要判断一个对象是否能被调用，能被调用的对象就是一个Callable对象，比如函数和我们上面定义的带有__call()__的类实例：
```
callable(Student())
# True
```

## 使用元类

### type()

type()函数可以查看一个类型或变量的类型，Hello是一个class，它的类型就是type，而h是一个实例，它的类型就是class Hello。

还可以创建一个新的类型。
要创建一个class对象，type()函数依次传入3个参数：
1.class的名称；
2.继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
3.class的方法名称与函数绑定，这里我们把函数fn绑定到方法名hello上。

```
def fn(self, name='world'): # 先定义函数
    print('Hello, %s.' % name)

 Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
```
通过`type()`数创建的类和直接写class是完全一样的，因为Python解释器遇到class定义时，仅仅是扫描一下class定义的语法，然后调用`type()`函数创建出class。

###　metaclass

`metaclass`直译为“元类”，用来控制类的创建行为。
创建类的流程应该是：先定义metaclass，就可以创建类，最后创建实例。

用到比较少，但是是Python面向对象里最难理解，也是最难使用的魔术代码。

定义metaclass时，metaclass的类名总是以Metaclass结尾，以便清楚地表示这是一个metaclass。

```
class MylistMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        attrs['dele'] = lambda self, value: self.remove(value)
        return type.__new__(cls, name, bases, attrs)


class Mylist(list):
    __metaclass__ = MylistMetaclass

a = Mylist()

a.add("a")

print a
a.dele("a")
print a

```
当我们写下`__metaclass__ = ListMetaclass`语句时，魔术就生效了，它指示Python解释器在创建MyList时，要通过`ListMetaclass.__new__()`来创建，在此，我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。

`__new__()`方法接收到的参数依次是：

1.当前准备创建的类的对象；

2.类的名字；

3.类继承的父类集合；

4.类的方法集合。

那么，动态修改的意义在哪？一般的写直接添加'add'放法即可。

但是，总会遇到需要通过`metaclass`修改类定义的。ORM就是一个典型的例子。

ORM全称“Object Relational Mapping”，即对象-关系映射，就是把关系数据库的一行映射为一个对象，也就是一个类对应一个表，这样，写代码更简单，不用直接操作SQL语句。

要编写一个ORM框架，所有的类都只能动态定义，因为只有使用者才能根据表的结构定义出对应的类来。

让我们来尝试编写一个ORM框架。

编写底层模块的第一步，就是先把调用接口写出来。比如，使用者如果使用这个ORM框架，想定义一个User类来操作对应的数据库表User，我们期待他写出这样的代码：
```
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

# 创建一个实例：
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库：
u.save()
```
首先来定义Field类，它负责保存数据库表的字段名和字段类型：
```
class Field(object):
    def __init__(self, name, column_type):
        self.name = name
        self.column_type = column_type

    def __str__(self):
        return '<%s:%s>' % (self.__class__.__name__, self.name)
```

然后，在此基础上进一步定义各类Field：
```
class StringField(Field):
    def __init__(self,name):
        super(StringField, self).__init__(name, 'varchar(100)')


class IntField(Field):
    def __init__(self,name):
        super(IntField,self).__init__(name, 'bigint')
```

接下来开始写`metaclass`：
```
class ModelMetaclass(type):
    def __new__(cls, name, bases, attrs):
        if name == 'Model':
            return type.__new__(cls, name, bases, attrs)
        print('Found model: %s' % name)
        mappings = dict()
        for k, v in attrs.iteritems():
            if isinstance(v, Field):
                print "Found mapping:%s ==> %s"%(k, v)
                mappings[k] = v
        for k in mappings.iterkeys():
            attrs.pop(k)
        attrs['__mappings__'] = mappings # 保存属性和列的映射关系
        attrs['__table__'] = name # 假设表名和类名一致
        return type.__new__(cls, name, bases, attrs)

```
`metaclass`一共做了三件事情：

1.排除对`Model`的修改；

2.在当前类（比如User）中查找定义的类的所有属性，如果找到一个Field属性，就把它保存到一个__mappings__的dict中，同时从类属性中删除该Field属性，否则，容易造成运行时错误；

3.把表名保存到__table__中，这里简化为表名默认为类名。

再接下来，创建基类Model：
```
class Model(dict):
    __metaclass__ = ModelMetaclass

    def __init__(self, **kw):
        super(Model, self).__init__(**kw)

    def __getattr__(self, key):
        try:
            return self[key]
        except KeyError:
            raise AttributeError(r"'Model' object has no attribute '%s'" % key)

    def __setattr__(self, key, value):
        self[key] = value

    def save(self):
        fields = []
        params = []
        args = []
        for k, v in self.__mappings__.iteritems():
            fields.append(v.name)
            params.append('?')
            args.append(getattr(self, k, None))
        sql = 'insert into %s (%s) values (%s)' % (self.__table__, ','.join(fields), ','.join(params))
        print('SQL: %s' % sql)
        print('ARGS: %s' % str(args))
        print('table name:%s'%self.__table__)
```

当用户定义一个`class User(Model)`时，Python解释器首先在当前类User的定义中查找`__metaclass__`，如果没有找到，就继续在父类Model中查找`__metaclass__`，找到了，就使用`Model`中定义的`__metaclass__`的`ModelMetaclass`来创建`User`类，也就是说，`metaclass`可以隐式地继承到子类，但却在子类处没有显示。


在接下来实例化User：
```
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
u.save()

#Found model: User
#Found mapping:email ==> <StringField:email>
#Found mapping:password ==> <StringField:password>
#Found mapping:id ==> <IntField:uid>
#Found mapping:name ==> <StringField:username>
#SQL: insert into User (password,email,username,uid) values (?,?,?,?)
#ARGS: ['my-pwd', 'test@orm.org', 'Michael', 12345]
#table name:User
```

在我们编写的ORM中，ModelMetaclass会删除掉User类的所有类属性，目的就是避免造成混淆。

>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
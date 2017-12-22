---
title: 初探python(9)IO编程
date: 2016-10-13 16:55:30
tags: 
- python
- python教程
categories: 总结
---


IO在计算机中指Input/Output，也就是输入和输出。由于程序和运行时数据是在内存中驻留，由CPU这个超快的计算核心来执行，涉及到数据交换的地方，通常是磁盘、网络等，就需要IO接口。
<!-- more -->
IO编程中，Stream（流）是一个很重要的概念，可以把流想象成一个水管，数据就是水管里的水，但是只能单向流动。Input Stream就是数据从外面（磁盘、网络）流进内存，Output Stream就是数据从内存流到外面去。对于浏览网页来说，浏览器和新浪服务器之间至少需要建立两根水管，才可以既能发数据，又能收数据。

由于CPU和内存的速度远远高于外设的速度，所以，在IO编程中，就存在速度严重不匹配的问题。也就是说，输出快，输入慢。有两种解决办法：

1.第一种是CPU等着，也就是程序暂停执行后续代码，等100M的数据在10秒后写入磁盘，再接着往下执行，这种模式称为同步IO；

2.是CPU不等待，只是告诉磁盘，“您老慢慢写，不着急，我接着干别的事去了”，于是，后续代码可以立刻接着执行，这种模式称为异步IO。

很明显，同步IO性能远远低于异步IO，但异步IO的缺点就是编程模型复杂。想想看，你得知道什么时候通知你“汉堡做好了”，而通知你的方法也各不相同。如果是服务员跑过来找到你，这是回调模式，如果服务员发短信通知你，你就得不停地检查手机，这是轮询模式。总之，异步IO的复杂度远远高于同步IO。

本章内容都是同步IO模式。

## 文件读写
现代操作系统不允许普通的程序直接操作磁盘，所以，读写文件就是请求操作系统打开一个文件对象（通常称为文件描述符），然后，通过操作系统提供的接口从这个文件对象中读取数据（读文件），或者把数据写入这个文件对象（写文件）。

### 读文件
要以读文件的模式打开一个文件对象，使用Python内置的`open()`函数，传入文件名和标示符，标示符'r'表示读，这样，我们就成功地打开了一个文件。
读取二进制文件，比如图片，视频等，用'rb'模式打开即可。
```
try:
    f = open('/path/to/file', 'r')
    print f.read()
finally:
    if f:
        f.close()
```
一旦文件读写失败会报一个`IOerror`，一旦出错，后面的f.close()就不会调用。所以我们要将其用`try ... finally`包括起来。

但为了方便，Python引入了`with`语句来自动帮我们调用`close()`方法:
```
with open('/path/to/file', 'r') as f:
    print f.read()
```
`read()`可以一次性读取到文件的全部内容。如果文件有10G，内存就爆了，所以，要保险起见，可以反复调用`read(size)`方法，每次最多读取size个字节的内容。另外，调用`readline()`可以每次读取一行内容，调用`readlines()`一次读取所有内容并按行返回list。因此，要根据需要决定怎么调用。

如果文件很小，read()一次性读取最方便；如果不能确定文件大小，反复调用read(size)比较保险；如果是配置文件，调用readlines()最方便：
```
for line in f.readlines():
    print(line.strip()) # 把末尾的'\n'删掉
```
###　file-like Object
像open()函数返回的这种有个read()方法的对象，在Python中统称为file-like Object。

`StringIO`就是在内存中创建的file-like Object，常用作临时缓冲。


读取二进制文件，比如图片，视频等，用'rb'模式打开即可。

### 字符编码
要读取非ASCII编码的文本文件，就必须以二进制模式打开，再解码。比如GBK编码的文件：
```
f = open('/Users/michael/gbk.txt', 'rb')
u = f.read().decode('gbk')
u
#u'\u6d4b\u8bd5'
print u
#测试
```
也可以直接导入`codecs`模块，帮我们自动转化编码。
```
import codecs
with codecs.open('/Users/michael/gbk.txt', 'r', 'gbk') as f:
    f.read() # u'\u6d4b\u8bd5'
```
### 写文件

写文件和读文件是一样的，唯一区别是调用open()函数时，传入标识符'w'或者'wb'表示写文本文件或写二进制文件。
```
import codecs
with codecs.open('/Users/michael/gbk.txt', 'r', 'gbk') as f:
    f.write('nihao')
```

## 操作文件和目录

如果我们在python下要操作文件、目录，可以使用`os`模块。
我们可以使用'os.name'来显示系统名字，如果是`posix`，说明系统是Linux、Unix或Mac OS X，如果是`nt`，就是Windows系统。

如果要获取详细信息，可以调用`uname()`函数。但在Windows下不支持。

###　环境变量
环境变量全部保存在`os.eniron`中。
要获取某个环境变量的值，可以调用`os.getenv()`函数。例如：`os.getenv('PATH')`

###　操作文件和目录

操作文件和目录的函数一部分放在os模块中，一部分放在os.path模块中，这一点要注意一下。

```
# 查看当前目录的绝对路径:
>>> os.path.abspath('.')
'/Users/michael'
# 在某个目录下创建一个新目录，
# 首先把新目录的完整路径表示出来:
>>> os.path.join('/Users/michael', 'testdir')
'/Users/michael/testdir'
# 然后创建一个目录:
>>> os.mkdir('/Users/michael/testdir')
# 删掉一个目录:
>>> os.rmdir('/Users/michael/testdir')
# 对文件重命名:
>>> os.rename('test.txt', 'test.py')
# 删掉文件:
>>> os.remove('test.py')

```
把两个路径合为一个，用os.path.join()会自动将两个路径相连，并符合系统要求。无需自己手写字符串相连。

同样的道理，要拆分路径时，也不要直接去拆字符串，而要通过`os.path.split()`函数，这样可以把一个路径拆分为两部分，后一部分总是最后级别的目录或文件名。

`os.path.splitext()`可以直接让你得到文件扩展名，很多时候非常方便。

这些合并和拆分不要求真实存在，只对字符串进行操作。

但复制操作并不在`os`模块中。原因是复制文件并非由操作系统提供的系统调用。

幸运的是`shutil`模块提供了`copyfile()`的函数，你还可以在`shutil`模块中找到很多实用函数，它们可以看做是os模块的补充。
```
import os
import shutil
s = os.path.abspath('.')
t = os.path.join(s, 'wode.txt')
print t
s = os.path.join(s,"woqu.txt")
shutil.copyfile(t,s)
```

最后，还可以通过python的特性来过滤文件。
```
# 列出所有该目录下的文件夹（即目录）
[x for x in os.listdir('.') if os.path.isdir(x)]
# 列出所有该目录下的.py文件
[x for x in os.listdir('.') if os.path.isfile(x) and os.path.splitext(x)[1]=='.py']

```

### 练习
题目：编写一个search(s)的函数，能在当前目录以及当前目录的所有子目录下查找文件名包含指定字符串的文件，并打印出完整路径。
```
__author__ = 'zjbao123'
'''编写一个search(s)的函数，
能在当前目录以及当前目录的所有子目录下查找文件名包含指定字符串的文件，
并打印出完整路径

Usage:
    > search.py str
    str: the str what you want to search in filename in your dir
'''
import os
import sys
def search(s):
    for root,dirs,files in os.walk('.'):
        for f in files:
            if(f.find(s) != -1):
                print os.path.join(root,f)[2:]

if __name__ == '__main__':
    search(sys.argv[1])
```

##　序列化
在程序运行的过程中，所有的变量都是在内存中，一旦程序结束，变量所占用的内存就被操作系统全部回收。即便你有所改动，没有被存储到磁盘里的，下次运行又回到了初始值。

我们把变量从内存中变成可存储或传输的过程称之为序列化，在Python中叫`pickling`，在其他语言中也被称之为`serialization`，`marshalling`，`flattening`等等，都是一个意思。

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即`unpickling`。

Python提供两个模块来实现序列化：`cPickle`和`pickle`。这两个模块功能是一样的，区别在于`cPickle`是C语言写的，速度快，pickle是纯Python写的，速度慢，跟`cStringIO`和`StringIO`一个道理。用的时候，先尝试导入`cPickle`，如果失败，再导入`pickle`：
```
try:
    import cPickle as pickle
except ImportError:
    import pickle

#序列化
d = dict(name='bob', sex='m', age='19')
with open('wode.txt','w') as f:
    pickle.dump(d, f)

#反序列化
with open('wode.txt', 'r') as f:
    w = pickle.load(f)
    print w
```

Pickle的问题和所有其他编程语言特有的序列化问题一样，就是它只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，不能成功地反序列化也没关系。

###　JSON

如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如XML，但更好的方法是序列化为JSON，因为JSON表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。JSON不仅是标准格式，并且比XML更快，而且可以直接在Web页面中读取，非常方便。

JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：

|JSON类型	|Python类型
|:-----------|:---------|
|{}|	dict|
|[]|	list|
|"string"|	'str'或u'unicode'|
|1234.56	|int或float|
|true/false	|True/False|
|null	|None|

Python内置的`json`模块提供了非常完善的Python对象到JSON格式的转换。
```
d = dict(name='bob', sex='m', age='19')
s=json.dumps(d)
w = json.loads(s)
print w
```
有一点需要注意，就是反序列化得到的所有字符串对象默认都是unicode而不是str。由于JSON标准规定JSON编码是UTF-8，所以我们总是能正确地在Python的str或unicode与JSON的字符串之间转换。

python 3 很好的解决了这个问题。

### JSON进阶
我们大多数时候喜欢用class表示对象，那来讲一讲如何序列化类：
```
import json

class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)
print(json.dumps(s))
```
但是直接调用上述代码会发现，报错了。原因是说Student对象不是一个可序列化为JSON的对象。dumps()方法不知道如何将Student实例变为一个JSON的{}对象。

可选参数default就是把任意一个对象变成一个可序列为JSON的对象，因此我们需要另外写一个函数将函数值传入。
```
def student2dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }

print(json.dumps(s, default=student2dict))
```
但还有一个偷懒的写法：

```
print(json.dumps(s, default=lambda obj: obj.__dict__))
```
因为通常class的实例都有一个`__dict__`属性，它就是一个dict，用来存储实例变量。也有少数例外，比如定义了`__slots__`的class。

因此，Python语言特定的序列化模块是pickle，但如果要把序列化搞得更通用、更符合Web标准，就可以使用json模块。

>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
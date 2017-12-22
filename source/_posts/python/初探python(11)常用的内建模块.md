---
title: 初探python(11)常用的内建模块
date: 2016-11-06 21:26:30
tags: 
- python
- python教程
categories: 总结
---
Python之所以自称“batteries included”，就是因为内置了许多非常有用的模块，无需额外安装和配置，即可直接使用。下面来介绍一下。
## collections

collections是Python内建的一个集合模块，提供许多有用的集合类。
<!-- more -->
### namedtuple

在之前的学习中我们知道tuple可以表示不变集合，例如，一个点的二维坐标就可以表示成`p = (1, 2)`,但是光光看到这个`(1,2)`，很难看出是用来表示一个坐标的。
定义一个class又小题大做。

这时，我们需要用到`namedtuple`：
```
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
p.x
# 1
p.y
# 2
```
`namedtuple`是一个函数，用来创建一个自定义的`tuple`对象，并且规定了`tuple`元素的个数，并可以用属性而不是索引来引用`tuple`的某个元素。

这样一来，我们用`namedtuple`可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便。

输入`help(namedtuple)`你会发现，
```
>>> d = p._asdict()                 # convert to a dictionary
>>> d['x']
11
>>> Point(**d)                      # convert from a dictionary
Point(x=11, y=22)
>>> p._replace(x=100)               # _replace() is like str.replace() but targets named fields
Point(x=100, y=22)

```
在帮助文档中，也提供了tuple转字典的方法，以及一些替换的方法。
### deque

之前在学习的时候已经碰到了这个函数，看了下官方解释：

```
deque([iterable[, maxlen]])
Build an ordered collection with optimized access from its endpoints.
```
建立一个从两个端点进行优化的有序集合。

简而言之，是一个可以设置最大长度的队列或堆，优点是`deque`除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素；此外，还可以用来取最近的几条数据，后来的数据会顶开早些的数据。

### defaultdict

与`dict`相比，不同之处在于如果key不存在时，返回一个默认值。
注意：默认值是调用函数返回的，而函数在创建`defaultdict`对象时传入。
```
dd = defaultdict(lambda: 'N/A')

```
除了在`Key`不存在时返回默认值，`defaultdict`的其他行为跟`dict`是完全一样的。

### OrderedDict

正如名字所言，有序字典。他的有序指的是Key会按照插入的顺序排列，不是Key本身排序。

### Counter

Counter是一个简单的计数器，例如，统计字符出现的个数：
```
from collections import Counter
c = Counter()
for ch in 'programming':
...     c[ch] = c[ch] + 1
...
c
#Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})

```
Counter实际上也是dict的一个子类。

## base64
Base64是一种任意二进制到文本字符串的编码方法，常用于在URL、Cookie、网页中传输少量二进制数据。

### 原理

首先，准备一个包含64个字符的数组：
```
['A', 'B', 'C', ... 'a', 'b', 'c', ... '0', '1', ... '+', '/']
```
然后，对二进制数据进行处理，每3个字节一组，一共是3x8=24bit，划为4组，每组正好6个bit,也就是2^6,64个字符相对应。

这样我们得到4个数字作为索引，然后查表，获得相应的4个字符，就是编码后的字符串。

Base64编码会把3字节的二进制数据编码为4字节的文本数据，长度增加33%，好处是编码后的文本数据可以在邮件正文、网页等直接显示。

如果要编码的二进制数据不是3的倍数，最后会剩下1个或2个字节怎么办？Base64用\x00字节在末尾补足后，再在编码的末尾加上1个或2个=号，表示补了多少字节，解码的时候，会自动去掉。

```
import base64
base64.b64encode('binary\x00string')
# 对于在URL，有一种"url safe"的base64编码，
# 把字符+和/分别变成-和_，防止搞混
base64.urlsafe_b64decode('abcd--__')
```
由于`=`字符也可能出现在Base64编码中，但`=`用在URL、Cookie里面会造成歧义，所以，很多Base64编码后会把`=`去掉,但在解码的时候，我们需要把`=`加上。
```
def b64decode_self(str):
    return base64.b64decode(str+'='*(4-len(str)%4))
```

## struct
用来解决str和其他二进制数据类型的转换。

`struct`的`pack`函数把任意数据类型变成字符串。
```
import struct
struct.pack('>I', 10240099)
# '\x00\x9c@c'
```
pack的第一个参数是处理指令，'>I'的意思是：

`>`表示字节顺序是big-endian，大端字节序也就是网络序，高字节存于内存低地址；低字节存于内存高地址。I表示4字节无符号整数。

`unpack`把`str`变成相应的数据类型：
```
 struct.unpack('>IH', '\xf0\xf0\xf0\xf0\x80\x80')
(4042322160, 32896)
```
具体处理指令可用`help`来查看。

下面我们用`struct`分析一下bmp格式。
首先找一个`bmp`文件，没有的话用“画图”画一个。
读入前30个字节来分析。

BMP格式采用小端方式存储数据，文件头的结构按顺序如下：

两个字节：'BM'表示`Windows`位图，'BA'表示OS/2位图；
一个4字节整数：表示位图大小；
一个4字节整数：保留位，始终为0；
一个4字节整数：实际图像的偏移量；
一个4字节整数：Header的字节数；
一个4字节整数：图像宽度；
一个4字节整数：图像高度；
一个2字节整数：始终为1；
一个2字节整数：颜色数。

所以，组合起来用unpack读取：
```
struct.unpack('<ccIIIIIIHH', s)
('B', 'M', 691256, 0, 54, 40, 640, 360, 1, 24)
```
结果显示，'B'、'M'说明是Windows位图，位图大小为640x360，颜色数为24。


### 练习

请编写一个bmpinfo.py，可以检查任意文件是否是位图文件，如果是，打印出图片大小和颜色数。
```
import struct


def isbmp(s):
    f = open(s, 'rb').read(30)
    b = struct.unpack('<ccIIIIIIHH', f)
    if b[0] == 'B' and b[1] == 'M':
        print "size = %s * %s" % (b[6], b[7])
        print "color = %s " % (b[-1])
    else:
        print "it's not bmp!"
if __name__ == '__main__':
    isbmp('1.bmp')
    isbmp("1.jpg")
```

## hashlib

摘要算法在很多地方都有广泛的应用。要注意摘要算法不是加密算法，不能用于加密（因为无法通过摘要反推明文），只能用于防篡改，但是它的单向计算特性决定了可以在不存储明文口令的情况下验证用户口令。

Python的hashlib提供了常见的摘要算法，如MD5，SHA1等等。

什么是摘要算法呢？摘要算法又称哈希算法、散列算法。它通过一个函数，把任意长度的数据转换为一个长度固定的数据串（通常用16进制的字符串表示）。

MD5是最常见的摘要算法，速度很快，生成结果是固定的128 bit字节，通常用一个32位的16进制字符串表示。
```
import hashlib

md5 = hashlib.md5()
md5.update('how to use md5 in python hashlib?')
print md5.hexdigest()
```
而SHA1也是其中一种算法，和调用MD5完全类似:
```
import hashlib

sha1 = hashlib.sha1()
sha1.update('how to use sha1 in ')
sha1.update('python hashlib?')
print sha1.hexdigest()
```
比SHA1更安全的算法是SHA256和SHA512，不过越安全的算法越慢，而且摘要长度更长。

有没有可能两个不同的数据通过某个摘要算法得到了相同的摘要？完全有可能。因为任何摘要算法都是把无限多的数据集合映射到一个有限的集合中。这种情况称为碰撞。
### 应用
这种用法常用于数据保密。如明文保存账户数据容易被黑客盗取，用户登录时，通过用户输入的口令来计算摘要，如果和存储的摘要一致，就可以认为是登录正确。

存储MD5的好处是即使运维人员能访问数据库，也无法获知用户的明文口令。

采用MD5存储口令是否就一定安全呢？也不一定。假设你是一个黑客，已经拿到了存储MD5口令的数据库，如何通过MD5反推用户的明文口令呢？暴力破解费事费力，真正的黑客不会这么干。

考虑这么个情况，很多用户喜欢用123456，888888，password这些简单的口令，于是，黑客可以事先计算出这些常用口令的MD5值，得到一个反推表：
```
'e10adc3949ba59abbe56e057f20f883e': '123456'
'21218cca77804d2ba1922c33e0151105': '888888'
'5f4dcc3b5aa765d61d8327deb882cf99': 'password'
```
这样，无需破解，只需要对比数据库的MD5，黑客就获得了使用常用口令的用户账号。

对于用户来讲，当然不要使用过于简单的口令。但是，我们能否在程序设计上对简单口令加强保护呢？

### 加盐
由于常用口令的MD5值很容易被计算出来，所以，要确保存储的用户口令不是那些已经被计算出来的常用口令的MD5，这一方法通过对原始口令加一个复杂字符串来实现，俗称“加盐”：

```
def calc_md5(password):
    return get_md5(password + 'the-Salt')
```
经过Salt处理的MD5口令，只要Salt不被黑客知道，即使用户输入简单口令，也很难通过MD5反推明文口令。

但是如果有两个用户都使用了相同的简单口令比如123456，在数据库中，将存储两条相同的MD5值，这说明这两个用户的口令是一样的。有没有办法让使用相同口令的用户存储不同的MD5呢？

如果假定用户无法修改登录名，就可以通过把登录名作为Salt的一部分来计算MD5，从而实现相同口令的用户也存储不同的MD5。


### 练习
1.设计一个验证用户登录小程序，根据用户输入的口令保存账号密码，亦可做登录检查。

解：
```
import hashlib
def md5(str):
    md = hashlib.md5()
    md.update(str)
    md_5 = md.hexdigest()
    return md_5
dict_client = {}
while 1:
    ty = raw_input("欢迎使用用户登录系统...存储用户数据请按1，登录请按2，退出请按3\n")
    if ty =='1':

        name_str = raw_input("请输出用户名字：\n")
        keywords_str = raw_input("请输入密码：\n")
        dict_client[name_str]=md5(keywords_str)
        print 'MD5密码加密已完成...'
        print dict_client
    if ty =="2":
        name_str = raw_input("请输出用户名字：\n")
        keywords_str = raw_input("请输入密码：\n")
        if dict_client[name_str]==md5(keywords_str):
            print "welcome %s" %name_str
        else :
            print "your keywords are wrong ! "
    if ty =="3":
        print 'BYE...'
        break
```
2.根据用户输入的登录名和口令模拟用户注册，计算更安全的MD5.
```
db = {}

def register(username, password):
    db[username] = get_md5(password + username + 'the-Salt')

```
然后，根据修改后的MD5算法实现用户登录的验证：
```
def login(username, password):
    pass
```
解法：
```
import hashlib


def md5(str):
    md = hashlib.md5()
    md.update(str)
    md_5 = md.hexdigest()
    return md_5


def register(username, password):
    dict_client[username] = md5(password + username + 'the-Salt')


def login(username, password):
    if dict_client[username] == md5(password + username + 'the-Salt'):
        print "welcome %s" % username
    else:
        print "your keywords are wrong ! "


dict_client = {}
while 1:
    ty = raw_input("欢迎使用用户登录系统...存储用户数据请按1，登录请按2，退出请按3\n")
    if ty == '1':
        name_str = raw_input("请输出用户名字：\n")
        keywords_str = raw_input("请输入密码：\n")
        register(name_str, keywords_str)
        print 'MD5密码加密已完成...'
        print dict_client
    if ty == "2":
        name_str = raw_input("请输出用户名字：\n")
        keywords_str = raw_input("请输入密码：\n")
        login(name_str,keywords_str)
    if ty == "3":
        print 'BYE...'
        break
```
## itertools

`itertools`提供了非常有用的用于操作迭代对象的函数。

### count
```
natuals = itertools.count(1,2)
for n in natuals:
    print n
```

返回一个从第一个参数1开始的连续值，可以设置第二个参数`step`步长。

### cycle()
```
cs = itertools.cycle("ABCD")
```
Return elements from the iterable until it is exhausted.
当然。电脑不会精疲力竭，将传入的序列无限重复下去。
### repeat()
```
ns = itertools.repeat('A', 10)
```
repeat(object [,times]) -> create an iterator which returns the object
for the specified number of times.  If not specified, returns the object
endlessly.

负责把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数,不提供，同样无穷重复。

### takewhile()
无限序列虽然可以无限迭代下去，但是通常我们会通过`takewhile()`等函数根据条件判断来截取一个有限序列。

```
import itertools

ns = itertools.count(1)

n = itertools.takewhile(lambda x: x <= 10, ns)
for i in n:
    print i
```

`itertools`提供的几个迭代器操作函数更加有用：

### chain()
```
for c in itertools.chain('ABC', 'XYZ'):
    print c
```
加强版的`cycle`，可以将一组迭代对象串联起来，形成一个更大的迭代器。

### groupby()
把迭代器中相邻的重复元素挑出来放在一起。

实际上挑选规则是通过函数完成的，只要作用于函数的两个元素返回的值相等，这两个元素就被认为是在一组的，而函数返回值作为组的key,第二个参数用来筛选规则例如可以忽略大小写。
```
for key, group in itertools.groupby('AAABBBbCCAAAaaa',lambda a:a.upper()):
    print key, list(group) 
```

### imap()
`imap()`和`map()`的区别在于，`imap()`可以作用于无穷序列，并且，如果两个序列的长度不一致，以短的那个为准。
长与短序列对应相乘。
```
for x in itertools.imap(lambda x, y: x * y, [10, 20, 30], itertools.count(1)):
    print x
for n in itertools.imap(lambda x:x*x,[1,2,3]):
    print n

```
注意，`imap()`返回一个迭代对象，而`map()`返回一个list，当你调用map事，已经计算完毕。而当你调用imap时，并没有进行任何计算。必须用for循环对其进行迭代，才会在每次循环过程中计算出下一个元素。

换种说法，`imap()`是`map()`的惰性实现。

同理，ifilter()就是filter()的惰性实现。
还有islice()，也是slice()的惰性实现。

## XML

XML虽然比JSON复杂，在Web中应用也不如以前多了，不过仍有很多地方在用，所以，有必要了解如何操作XML。

### DOM vs SAX

操作XML有两种方法：DOM和SAX。DOM会把整个XML读入内存，解析为树，因此占用内存大，解析慢，优点是可以任意遍历树的节点。SAX是流模式，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件。

正常情况下，优先考虑SAX，因为DOM实在太占内存。

在Python中使用SAX解析XML非常简洁，通常我们关心的事件是`start_element`，`end_element`和`char_data`，准备好这3个函数，然后就可以解析xml了。

举个例子，当SAX解析器读到一个节点时：
```
<a href="/">python</a>
```
会产生3个事件：
1.start_element事件，在读取<a href="/">时；
2.char_data事件，在读取python时；
3.end_element事件，在读取</a>时。

```
from xml.parsers.expat import ParserCreate

class DefaultSaxHandler(object):
    def start_element(self, name, attrs):
        print('sax:start_element: %s, attrs: %s' % (name, str(attrs)))

    def end_element(self, name):
        print('sax:end_element: %s' % name)

    def char_data(self, text):
        print('sax:char_data: %s' % text)

xml = r'''<?xml version="1.0"?>
<ol>
    <li><a href="/python">Python</a></li>
    <li><a href="/ruby">Ruby</a></li>
</ol>
'''
handler = DefaultSaxHandler()
parser = ParserCreate()
parser.returns_unicode = True
parser.StartElementHandler = handler.start_element
parser.EndElementHandler = handler.end_element
parser.CharacterDataHandler = handler.char_data
parser.Parse(xml)
```
而生成XML可以使用`append()`和`join()`拼接字符串的方式。

如果生成复杂的XML呢？建议你不要用XML，改成JSON。

## HTMLParser
一个便于解析HTML的模块。

如果我们要编写一个搜索引擎，第一步是用爬虫把目标网站的页面抓下来，第二步就是解析该HTML页面，看看里面的内容到底是新闻、图片还是视频。

假设第一步已经完成了，第二步应该如何解析HTML呢？

HTML本质上是XML的子集，但是HTML的语法没有XML那么严格，所以不能用标准的DOM或SAX来解析HTML。
```
from HTMLParser import HTMLParser
from htmlentitydefs import name2codepoint

class MyHTMLParser(HTMLParser):

    def handle_starttag(self, tag, attrs):
        print('<%s>' % tag)

    def handle_endtag(self, tag):
        print('</%s>' % tag)

    def handle_startendtag(self, tag, attrs):
        print('<%s/>' % tag)

    def handle_data(self, data):
        print('data')

    def handle_comment(self, data):
        print('<!-- -->')

    def handle_entityref(self, name):
        print('&%s;' % name)

    def handle_charref(self, name):
        print('&#%s;' % name)

parser = MyHTMLParser()
parser.feed('<html><head></head><body><p>Some <a href=\"#\">html</a> tutorial...<br>END</p></body></html>')
```

`feed()`方法可以多次调用，也就是不一定一次把整个HTML字符串都塞进去，可以一部分一部分塞进去。

特殊字符有两种，一种是英文表示的`&nbsp;`（占位符），一种是数字表示的`&#1234;`，这两种字符都可以通过Parser解析出来。

>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)和官方文档的一个学习内容的总结。以便于自己后的学习和整理。
---
title: python大闯关(0-4)
date: 2016-10-18 20:51:02
tags:
- python
categories: 总结/
---

最近在找python对应的学习实例时,看到了一个古老的游戏,方便大家边玩边学：

当当当当，**python challenge**！

[入口](http://www.pythonchallenge.com/)

当然，他是有规则的。
General tips:
* Use the hints. They are helpful, most of the times.
* Investigate the data given to you.
* Avoid looking for spoilers.
<!--more-->

谜一样的游戏，相当有趣，试试看自己能闯几关吧。


[repo](https://github.com/zjbao123/python_challenge_solution)
## 第0关
![第0关](http://www.pythonchallenge.com/pc/def/calc.jpg)
根据图片，就可以知道，2的38次方。
```
print (2**38)
```
计算得到结果,274877906944,输入即可到下一关。

## 第1关

![第1关](http://www.pythonchallenge.com/pc/def/map.jpg)

下面有一句提示：`everybody thinks twice before solving this.`
然后一堆乱码：

g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj.

从图中可以看出，每个字母都向后移了两位得到那个数字。这估计就是关键所在了。

思路是这样的：

首先，将26个字母的ASCII码表示出来，转换为对应的数字。a~z对应为97~122
将对应的数字移位解码输出，如ASCII码不在此范围则直接输出。

ASCII码对应的转化函数为：`chr(65) ord('a')`
```
def findLetter(s):
    sum =''
    for i in s:

        if 120>=ord(i)>=97:
             sum +=chr(ord(i)+2)
        elif 122>=ord(i)>=121:
            sum += chr(ord(i)-24)
        else:
            sum += i
    return sum
l = "g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj"
print findLetter(l)
```

解码结果为：

i hope you didnt translate it by hand. thats what computers are for. doing it in by hand is inefficient and that's why this text is so long. using string.maketrans() is recommended. now apply on the url

这边解码之后，又告诉我了另一个解码的方式，通过映射表来编码解码(怪自己知道的太少)：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import string

__author__ = 'zjbao123'
intab = 'abcdefghijklmnopqrstuvwxyz'
outtab = 'cdefghijklmnopqrstuvwxyzab'
trantab = string.maketrans(intab, outtab)

l = "map"
print l.translate(trantab)
```

输出结果一致。然后应用在map上，得到`equality`,进入第2关。

### 官方解答
只要将网址中的pc改为pcc即可看到官方解答.(第三关会告诉你这个秘密)

```
table = string.maketrans(string.ascii_lowercase,string.ascii_lowercase[2:]+string.ascii_lowercase[:2])
print string.translate(text,table)
```
尽管自己写的也挺简洁，但是有些方法还是不知道啊。

看样子手册这个东西是常用常新的。
```
for x in s:
    if ord(x)>=ord('a') and ord(x)<=ord('z'):
        o+=chr((ord(x)+2-ord('a'))%26+ord('a'))
    else:
        o+=x
print o
```
用笨办法也是写的非常简洁。
## 第2关
![第2关](http://www.pythonchallenge.com/pc/def/ocr.jpg)

recognize the characters. maybe they are in the book, 
but MAYBE they are in the page source.

题目看的不是很懂。查了一下对应的意思，答案应该是在源代码里，结果`F12`一看，一大段的乱码,开头有一句：

`find rare characters in the mess below:`

估计要爬网页了。


正好这几天在学爬虫，拿来练练手。有些投机取巧吧，猜到是数字或者字母了，写的惨不忍睹。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'

import urllib2, re

url = 'http://www.pythonchallenge.com/pc/def/ocr.html'
response = urllib2.urlopen(url)
content = response.read().decode('utf-8')
pattern = re.compile('-->(.*?)-->', re.S)
match = pattern.search(content)
result = match.group()
pattern2 = re.compile('\n')
result = pattern2.sub('', result)
pattern2 = re.compile('[a-zA-Z0-9]')
str = ('').join(pattern2.findall(result))

print str
```
最后输出：`equality`。
当然，删除回车，直接`replace("\n","")`也行，不用向我这么愚。

在得知是字母之后，源代码可以改良很多：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'

import urllib2, re

url = 'http://www.pythonchallenge.com/pc/def/ocr.html'
response = urllib2.urlopen(url)
content = response.read().decode('utf-8')
pattern = re.compile('<!--(.*?)-->', re.S)
match = pattern.findall(content)
a = [i for i in match[1] if i.isalpha()]
print(a) 
```

在得出正确答案之后，便去找了好一点的解法。一查发现，各种解法都有，唉，都怪自己学艺不精，连`count()`都没想到。在原基础上进行了改进。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'

import urllib2, re

url = 'http://www.pythonchallenge.com/pc/def/ocr.html'
response = urllib2.urlopen(url)
content = response.read().decode('utf-8')
pattern = re.compile('<!--(.*?)-->', re.S)
match = pattern.findall(content)

str = []
for i in match[1]:
    if i in str:
        pass
    else:
        str.append(i)
        print i + ' ', match[1].count(i)
print "".join(ch for ch in guts if d[ch] == 1)
```
这个问题在于没有直接输出答案，而且之前还觉得不错的`count()`效率并不高，就是根据题目意思来了，真的记了一个数，不过我觉得我之前的代码也没有太大问题，本来问题就没有唯一解。

### 官方解答
再来看看官方解答：
```
Rare characters = less frequent than average
```
一句话就说的很明白，就是找到低于平均数的值。
```
s = ''.join([line.rstrip() for line in open('ocr.txt')])    
OCCURRENCES = {}
for c in s: OCCURRENCES[c] = OCCURRENCES.get(c, 0) + 1
avgOC = len(s) // len(OCCURRENCES) #除法，四舍五入
print ''.join([c for c in s if OCCURRENCES[c] < avgOC])   
```
通过字典来获取值，这里用到了字典的一个方法：
* `D.get(k[,d])` 
 D[k] if k in D, else d.  d defaults to None.

另外，列表生成式可以用一行语句代替循环生成所需的list，作为python的一大高级特性应该好好利用。

这边官方给出了更进一步的解答：
```
data = ''.join([line.rstrip() for line in open('ocr.txt')])    
OCCURRENCES = collections.OrderedDict()
for c in data: OCCURRENCES[c] = OCCURRENCES.get(c, 0) + 1
avgOC = len(data) // len(OCCURRENCES)
print ''.join([c for c in OCCURRENCES if OCCURRENCES[c] < avgOC]) 
```
通过有序序列，使得结果有序输出，并且最后写列表生成式的时候循环次数明显减少，优化了这个过程。

在知道是字母之后，还有更加简单的写法，只需两行：
```
import string

text = ''.join([line.rstrip() for line in open('ocr.txt')])
print filter(lambda x: x in string.letters, text)
```
巧妙地用了匿名函数和过滤器，很python的程序。6的不行。自己还没活用到这种程度真的都不好意思说自己会python。

还有利用只出现一次这个特征来查找的，这个比之前网上找到的list方法好在巧用了字典和列表生成式。
```
guts = open("ocr.txt").read()
d = {}
for ch in guts:
    d[ch] = d.get(ch, 0) + 1
print "".join(ch for ch in guts if d[ch] == 1)
```
在之后也附上了一些网友的评价：
`count()`尽管效率不高O(n^2)但是还是运行的挺快，可读性强。需要在count之前加以判断，通过`has_key()`来进行优化。
```
text = ...   # the long string found in the source code of the page
output = "" # empty string
counts = {}
for c in text:
  if counts.has_key(c):
    continue
  counts[c] = text.count(c)
  if counts[c] < 100:  # guess that rare characters will occur less than 100 times
    output += c

print output
```

在之后的浏览中，又发现了一个很妙的写法。利用`set`这个无序，无重复的交集。
```
 print set("""<- lots of text ->""")
```
问题就在于无序，还是只找到了字母，没有连成单词。
```
guts = open("ocr.txt").read()
nb = set(guts)
d = dict()
for c in nb:
    d[c] = guts.count(c)
print ''.join( [c for c in guts if d[c] == 1 ] )
```

Who says python has only one way to do things?


接下来到了第3关。
##第3关

![第3关](http://www.pythonchallenge.com/pc/def/bodyguard.jpg)
One small letter, surrounded by EXACTLY three big bodyguards on each of its sides.

看完题目不知所云，好像是在说图上的蜡烛。但肯定不是，看了下源代码果然又要用正则。

不过看懂也就简单了，三个大写字母夹的那个小写字母(另外三个大写的两边要是小写)，就是所需的。两次正则就可以得到结果。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'

import urllib2, re

response = urllib2.urlopen('http://www.pythonchallenge.com/pc/def/equality.html')
content = response.read()
pattern = re.compile('<!--(.*?)-->', re.S)
match = pattern.findall(content)
with open('e.txt', 'w') as f:
    f.write(match[0])

pattern = re.compile('[^A-Z][A-Z]{3}([a-z])[A-Z]{3}[^A-Z]')
content =''.join((line.strip() for line in open('e.txt','r')))
match = pattern.findall(content)
print ''.join(i for i in match)
```
答案就是`linkedlist`
### 官方解答
利用列表生成式和递归的思想，简明扼要，
```
# text is in t2
markers = ''.join( [ '0' if c in string.lowercase else '1' for c in t2 ] )
def f( res, t2, markers ):
    n = len(markers.partition('011101110')[0])
    return f( res+t2[n+4], t2[n+9:], markers[n+9:] ) if n != len(markers) else res
print f( '', t2, markers )
```
上述例子还有一点要说明的是：当出现`HHHoHHHoHHHoHHHoHHH`时，只能匹配到第二个。因为第一个明显不符合规定，由于在开头，一开始就不是小写。之后的也因为位移过多而错过判断的值。因此我们需要对其进行改进。
```
def level_3(t1):
    # 因为没有用正则，我们需要伪装一下边界
    pad = "x"
    t1 = pad + t1 + pad
    # 检查是否为大写还是小写
    markers = "".join([ '1' if c in string.uppercase else '0' for c in t1])
    pattern = "011101110"
    def f(res, t2, markers):
        n = len(markers.partition(pattern)[0])
        # 出于性能考虑，我们本应该把这个lambda函数放在嵌套外，但出于可读性考虑，我觉得还是放在这吧
        end_of_string = lambda res, t2, markers: markers == pattern and res+t2[4] or res
        return f(res+t2[n+4], t2[n+4:], markers[n+4:]) if n != len(markers) else end_of_string(res, t2, markers)
    return f('', t1, markers)
```
或者不用lambda:
```
import string
def level_3(t1):
    # the second x ensures that the last match will never be in the last portion of the split after calling string.partition
    pad = "xx"
    t1 = pad + t1 + pad
    markers = "".join([ '1' if c in string.uppercase else '0' for c in t1])
    pattern = "011101110"
    def f(res, t2, markers):
        n = len(markers.partition(pattern)[0]
        return f(res+t2[n+4], t2[n+4:], markers[n+4:]) if n != len(markers) else res
    return f('', t1, markers)
```
另外，官方还介绍正则的用法。
一句话解决。

运用到了`(?:pattern)`来匹配 pattern 但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。这在使用 "或" 字符 (|) 来组合一个模式的各个部分是很有用。例如， 'industr(?:y|ies) 就是一个比 'industry|industries' 更简略的表达式。
```
 .join(re.findall('(?:^|[^A-Z])[A-Z]{3}([a-z])[A-Z]{3}(?:[^A-Z]|$)',text))
```
还有利用前瞻和后顾的：
```
 .join(re.findall("(?<=[^A-Z][A-Z]{3})[a-z](?=[A-Z]{3}[^A-Z])",text))
```
当然也有反向前瞻和后顾的：
```
pattern = re.compile(r'(?<\![A-Z])[A-Z]{3}([a-z])(?=[A-Z]{3}(?![A-Z]))') # !前没有斜杠
```

还有更吊的，直接把不匹配的变黑，需要的变白。这边只记录一下输出。

```
normal = '\033[0m'
black  = '\033[30;1m'
white  = '\033[37;1m'
for match in matches:
   print black + match[1:4] + white + match[4] + black + match[5:8] + normal
```


## 第4关
点击图片，显示`and the next nothing is 44827`，又修改源代码，发现只是数字不同而已。

这关比较简单，只要写一个循环就行。
```
__author__ = 'zjbao123'
import urllib2, re

url = 'http://www.pythonchallenge.com/pc/def/linkedlist.php?nothing='
tail = '37278'
while True:
    result = urllib2.urlopen(url + tail)
    str = result.read()
    pattern = re.compile("\d+")
    tail = pattern.search(str).group()
    print tail
```
稍等片刻，不过期间还是有点意思的，还搞了一些分岔路和一些误导的值，当然还需要自己手动判断一下。
最后得到答案：`peak.html`



## 小结
本来冲着过关来的，后来发现了“世外桃源”，官方写法才表现出了python的真正用法，不仅提供了python版，各个版本都有。自己写的轮子真的粗糙不堪，感觉自己的动力来自于看官方文档了呢。


好的，本期闯关先存个档，小伙伴们，我们下期见哈哈哈
---
title: python爬虫入门(2)
date: 2016-10-16 16:40:02
tags:
- python
- 爬虫
categories: 总结
---

## 异常处理

这里主要说的是URLError还有HTTPError，以及对它们的一些处理。

### URLError
首先解释下URLError可能产生的原因：

* 网络无连接，即本机无法上网
* 连接不到特定的服务器
* 服务器不存在

<!-- more -->
```
import urllib2
 
requset = urllib2.Request('http://www.xxxxx.com')
try:
    urllib2.urlopen(request)
except urllib2.URLError, e:
    print e.reason
```
当我们打开一个不存在的网址时，就会报错：`[Errno 11004] getaddrinfo failed`

### HTTPError
`HTTPError`是`URLError`的子类，在你利用`urlopen`方法发出一个请求时，服务器上都会对应一个应答对象`response`，其中它包含一个数字”状态码”。举个例子，假如`response`是一个”重定向”，需定位到别的地址获取文档，`urllib2`将对此进行处理。

其他不能处理的，`urlopen`会产生一个`HTTPError`，对应相应的状态吗，HTTP状态码表示HTTP协议所返回的响应的状态。下面将状态码归结如下：

>100：继续  客户端应当继续发送请求。客户端应当继续发送请求的剩余部分，或者如果请求已经完成，忽略这个响应。
101： 转换协议  在发送完这个响应最后的空行后，服务器将会切换到在Upgrade 消息头中定义的那些协议。只有在切换新的协议更有好处的时候才应该采取类似措施。
102：继续处理   由WebDAV（RFC 2518）扩展的状态码，代表处理将被继续执行。
200：请求成功      处理方式：获得响应的内容，进行处理
201：请求完成，结果是创建了新资源。新创建资源的URI可在响应的实体中得到    处理方式：爬虫中不会遇到
202：请求被接受，但处理尚未完成    处理方式：阻塞等待
204：服务器端已经实现了请求，但是没有返回新的信 息。如果客户是用户代理，则无须为此更新自身的文档视图。    处理方式：丢弃
300：该状态码不被HTTP/1.0的应用程序直接使用， 只是作为3XX类型回应的默认解释。存在多个可用的被请求资源。    处理方式：若程序中能够处理，则进行进一步处理，如果程序中不能处理，则丢弃
301：请求到的资源都会分配一个永久的URL，这样就可以在将来通过该URL来访问此资源    处理方式：重定向到分配的URL
302：请求到的资源在一个不同的URL处临时保存     处理方式：重定向到临时的URL
304：请求的资源未更新     处理方式：丢弃
400：非法请求     处理方式：丢弃
401：未授权     处理方式：丢弃
403：禁止     处理方式：丢弃
404：没有找到     处理方式：丢弃
500：服务器内部错误  服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。一般来说，这个问题都会在服务器端的源代码出现错误时出现。
501：服务器无法识别  服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求。
502：错误网关  作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
503：服务出错   由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。

HTTPError实例产生后会有一个code属性，这就是是服务器发送的相关错误号。
因为urllib2可以为你处理重定向，也就是3开头的代号可以被处理，并且100-299范围的号码指示成功，所以你只能看到400-599的错误号码。

下面我们写一个例子来感受一下，捕获的异常是HTTPError，它会带有一个code属性，就是错误代号，根据编程经验，父类的异常应当写到子类异常的后面，如果子类捕获不到，那么可以捕获父类的异常,还可以加入 hasattr属性提前对属性进行判断:
```
import urllib2
 
req = urllib2.Request('http://blog.csdn.net/cqcre')
try:
    urllib2.urlopen(req)
except urllib2.URLError, e:
    if hasattr(e,"code"):
        print e.code
    if hasattr(e,"reason"):
        print e.reason
else:
    print "OK"
```

## Cookie

Cookie，指某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据（通常经过加密）。

比如说有些网站需要登录后才能访问某个页面，在登录之前，你想抓取某个页面内容是不允许的。那么我们可以利用Urllib2库保存我们登录的Cookie，然后再抓取其他页面就达到目的了。

在此之前，先介绍Opener。
### Opener

Opener是urllib2.OpenerDirector的实例，在前面，我们都是使用的默认的opener，也就是urlopen。它是一个特殊的opener，可以理解成opener的一个特殊实例，传入的参数仅仅是url，data，timeout。

如果我们需要用到Cookie，只用这个opener是不能达到目的的，所以我们需要创建更一般的opener来实现对Cookie的设置。


### Cookielib

cookielib模块的主要作用是提供可存储cookie的对象，以便于与urllib2模块配合使用来访问Internet资源。Cookielib模块非常强大，我们可以利用本模块的CookieJar类的对象来捕获cookie并在后续连接请求时重新发送，比如可以实现模拟登录功能。该模块主要的对象有CookieJar、FileCookieJar、MozillaCookieJar、LWPCookieJar。

它们的关系：`CookieJar —-派生—->FileCookieJar  —-派生—–>MozillaCookieJar和LWPCookieJar`


#### 1)获取Cookie保存到变量
```
import urllib2
import cookielib
#声明一个CookieJar对象实例来保存cookie
cookie = cookielib.CookieJar()
#利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler = urllib2.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = urllib2.build_opener(handler)
#创建一个请求，原理同urllib2的urlopen
response = opener.open('http://www.baidu.com')

for item in cookie:
    print 'Name = ' + item.name
    print 'Value = '+ item.value
```

#### 2）保存Cookie到文件

在上面的方法中，我们将cookie保存到了cookie这个变量中，如果我们想将cookie保存到文件中该怎么做呢？这时，我们就要用到
FileCookieJar这个对象了，在这里我们使用它的子类MozillaCookieJar来实现Cookie的保存。
```
import urllib2
import cookielib
#设置保存cookie的文件，同级目录下的cookie.txt
filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookielib.MozillaCookieJar(filename)
#利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler = urllib2.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = urllib2.build_opener(handler)
#此处的open方法同urllib2的urlopen方法，也可以传入request
response = opener.open('http://www.baidu.com')

cookie.save(ignore_discard=True, ignore_expires=True)

```
>ignore_discard: save even cookies set to be discarded. 
ignore_expires: save even cookies that have expiredThe file is overwritten if it already exists


gnore_discard:即使cookies将被丢弃也将它保存下来。

ignore_expires：如果在该文件中cookies已经存在，则覆盖原文件写入。

在这里，我们将这两个全部设置为True。

#### 3)从文件中获取Cookie并访问
```
#创建MozillaCookieJar实例对象
cookie = cookielib.MozillaCookieJar()
#从文件中读取cookie内容到变量
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
#创建请求的request
req = urllib2.Request("http://www.baidu.com")
#利用urllib2的build_opener方法创建一个opener
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
response = opener.open(req)
print response.read()
```
#### 4)利用cookie模拟网站登录
```
import urllib
import urllib2
import cookielib
 
filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookielib.MozillaCookieJar(filename)
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
postdata = urllib.urlencode({
            'stuid':'201200131012',
            'pwd':'23342321'
        })
#登录教务系统的URL
loginUrl = 'http://jwxt.sdu.edu.cn:7890/pls/wwwbks/bks_login2.login'
#模拟登录，并把cookie保存到变量
result = opener.open(loginUrl,postdata)
#保存cookie到cookie.txt中
cookie.save(ignore_discard=True, ignore_expires=True)
#利用cookie请求访问另一个网址，此网址是成绩查询网址
gradeUrl = 'http://jwxt.sdu.edu.cn:7890/pls/wwwbks/bkscjcx.curscopre'
#请求访问成绩查询网址
result = opener.open(gradeUrl)
print result.read()
```

创建一个带有cookie的opener，在访问登录的URL时，将登录后的cookie保存下来，然后利用这个cookie来访问其他网址。






------------------------------------------------------------

## 参考

[如何入门 Python 爬虫？-知乎](https://www.zhihu.com/question/20899988)

[Python爬虫学习系列教程](http://cuiqingcai.com/1052.html)

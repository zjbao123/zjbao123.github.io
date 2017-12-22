---
title: python爬虫入门（1）
date: 2016-10-16 15:51:02
tags:
- python
- 爬虫
categories: 总结
---
## 前言

之前只是在学习Python的基本语法，并没有实战。时间所迫，就业所需，兴趣所驱，一边学习一边总结吧。又是一条漫长的打怪(bug)升级之路。
<!--more-->
## 什么是爬虫

### 爬虫的定义

把互联网比喻成一个蜘蛛网，那么Spider就是在网上爬来爬去的蜘蛛。
网络蜘蛛是通过网页的链接地址来寻找网页的。
从网站某一个页面（通常是首页）开始，读取网页的内容，找到在网页中的其它链接地址，
然后通过这些链接地址寻找下一个网页，每次看到一个可能需要爬的新链接，你就先查查你脑子里是不是已经去过这个页面地址。如果去过，那就别去了。这样一直循环下去，直到把这个网站所有的网页都抓取完为止。
如果把整个互联网当成一个网站，那么网络蜘蛛就可以用这个原理把互联网上所有的网页都抓取下来。
这样看来，网络爬虫就是一个爬行程序，一个抓取网页的程序。
网络爬虫的基本操作是抓取网页。

### 爬虫流程

* “所有网站皆可爬”

* 框架不变：**发送请求——获得页面——解析页面——下载内容——储存内容**

### 认识URI和URL

URI：Web上可用的每种资源 -HTML文档、图像、视频片段、程序等 - 由一个通用资源标识符（Uniform Resource Identifier, 简称"URI"）进行定位。

URL是Uniform Resource Locator的缩写，译为“统一资源定位符”。

URL是URI命名机制的一个子集。

URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。URI是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，所以，是绝对的，而通常说的relative URL，则是针对另一个absolute URL，本质上还是绝对的。

这里的相对绝对指的是是否有`协议:`这部分的内容，即访问该路径的方式。

示例：`协议://域名/目录a/目录b/文件c`

## Urllib库的使用


```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'
import urllib2
import urllib

user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36'
headers = {'User-Agent': user_agent, 'referer': 'https://www.zhihu.com/'}
values = {"username": "zjbao123", "password": "***"}
data = urllib.urlencode(values)
url = 'http://passport.csdn.net/account/login'
# 向服务器端发送请求
request = urllib2.Request(url, data, headers)
# 响应服务器的请求
response = urllib2.urlopen(request)
print response.read()
```
首先导入urllib2库。

* `urlopen(url, data, timeout)`
第一个参数url即为URL
第二个参数data是访问URL时要传送的数据,默认为None
第三个timeout是设置超时时间。默认为`socket._GLOBAL_DEFAULT_TIMEOUT`

* `urlencode(query, doseq=0)`
Encode a sequence of two-element tuples or dictionary into a URL query string.
将两个元素的元组或者是字典编码为URL查询字符串。

我这边传入的是一个`request`请求，然后用`response`来响应请求。通过`request`来传入所需的url和data。这边所需的data要通过`urllib.urlencode`来进行编码，输出一个URL查询字符串才可以传入。
这里使用的`POST`的方式进行传输。

### `POST`和`GET`
网页的数据传送分为`POST`和`GET`两种方式。

`GET`方式是直接以链接形式访问，链接中包含了所有的参数，当然如果包含了密码的话是一种不安全的选择，不过你可以直观地看到自己提交了什么内容。

而`POST`则不会在网址上显示所有的参数，不过如果你想直接查看提交了什么就不太方便了，大家可以酌情选择。

### 设置Headers

有些网站不会同意程序直接用上面的方式进行访问，如果识别有问题，那么站点根本不会响应，所以为了完全模拟浏览器的工作，所以，我们需要设置一些`Headers`的属性。

如果要查看你的`Headers`相应的属性，在浏览器界面按下`F12`-> `network` -> 选择一个文件 -> `Headers`

`user_agent`就是请求的身份，如果没有写入请求身份，那么服务器不一定会响应，所以可以在`headers`中设置`agent`,另外，我们还有对付”反盗链”的方式，对付防盗链，服务器会识别`headers`中的`referer`是不是它自己，如果不是，有的服务器不会响应，所以我们还可以在`headers`中加入`referer`。

同上面的方法，在传送请求时把headers传入Request参数里，这样就能应付防盗链了。

另外headers的一些属性，下面的需要特别注意一下：

>User-Agent : 有些服务器或 Proxy 会通过该值来判断是否是浏览器发出的请求
>Content-Type : 在使用 REST 接口时，服务器会检查该值，用来确定 HTTP Body 中的内容该怎样解析。
>application/xml ： 在 XML RPC，如 RESTful/SOAP 调用时使用
>application/json ： 在 JSON RPC 调用时使用
>application/x-www-form-urlencoded ： 浏览器提交 Web 表单时使用
>在使用服务器提供的 RESTful 或 SOAP 服务时， Content-Type 设置错误会导致服务器拒绝服务

### Proxy（代理）的设置

urllib2 默认会使用环境变量 http_proxy 来设置 HTTP Proxy。假如一个网站它会检测某一段时间某个IP 的访问次数，如果访问次数过多，它会禁止你的访问。所以你可以设置一些代理服务器来帮助你做工作，每隔一段时间换一个代理，网站君都不知道是谁在捣鬼了，这酸爽！

代理的设置用法：
```
enable_proxy = True
proxy_handler = urllib2.ProxyHandler({"http" : 'http://some-proxy.com:8080'})
null_proxy_handler = urllib2.ProxyHandler({})
if enable_proxy:
    opener = urllib2.build_opener(proxy_handler)
else:
    opener = urllib2.build_opener(null_proxy_handler)
urllib2.install_opener(opener)

```

### 使用 HTTP 的`PUT`和`DELETE`方法

http协议有六种请求方法，get,head,put,delete,post,options，我们有时候需要用到PUT方式或者DELETE方式请求。

>PUT：这个方法比较少见。HTML表单也不支持这个。本质上来讲， PUT和POST极为相似，都是向服务器发送数据，但它们之间有一个重要区别，PUT通常指定了资源的存放位置，而POST则没有，POST的数据存放位置由服务器自己决定。
>
>DELETE：删除某一个资源。基本上这个也很少见，不过还是有一些地方比如amazon的S3云服务里面就用的这个方法来删除资源。

```
import urllib2
request = urllib2.Request(uri, data=data)
request.get_method = lambda: 'PUT' # or 'DELETE'
response = urllib2.urlopen(request)
```

### 使用DebugLog

可以通过下面的方法把 Debug Log 打开，这样收发包的内容就会在屏幕上打印出来，方便调试，这个也不太常用，仅提一下。
```
import urllib2
httpHandler = urllib2.HTTPHandler(debuglevel=1)
httpsHandler = urllib2.HTTPSHandler(debuglevel=1)
opener = urllib2.build_opener(httpHandler, httpsHandler)
urllib2.install_opener(opener)
response = urllib2.urlopen('http://www.baidu.com')
```

------------------------------------------------------------

## 参考

[如何入门 Python 爬虫？-知乎](https://www.zhihu.com/question/20899988)

[Python爬虫学习系列教程](http://cuiqingcai.com/1052.html)



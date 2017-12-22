---
title: 初探python(12)web开发
date: 2016-09-13 22:14:50
tags: 
- python
- python教程
categories: 总结
---

## WSGI接口
在之前做的一些初步了解之后，我们明白，一个Web应用的本质就是：
1.浏览器发送一个HTTP请求；
2.服务器收到请求，生成一个HTML文档；
3.服务器把HTML文档作为HTTP响应的Body发送给浏览器；
4.浏览器收到HTTP响应，从HTTP Body取出HTML文档并显示。
<!-- more -->
所以，最简单的Web应用就是先把HTML用文件保存好，用一个现成的HTTP服务器软件，接收用户请求，从文件中读取HTML，返回。Apache、Nginx、Lighttpd等这些常见的静态服务器就是干这件事情的。

我们需要一个统一的接口，让我们专心用Python编写Web业务，而不是从底层开始一步步实现。当然，随着学习深入，自然需要这样走。目前我们所需的仅仅是一个接口。

这个接口就是WSGI：Web Server Gateway Interface。

WSGI接口定义非常简单，它只要求Web开发者实现一个函数，就可以响应HTTP请求。我们来看一个最简单的Web版本的“Hello, web!”：
```
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return '<h1>Hello, web!</h1>'
```

上面的`application()`函数就是符合WSGI标准的一个HTTP处理函数，它接收两个参数：

* environ：一个包含所有HTTP请求信息的dict对象；

* start_response：一个发送HTTP响应的函数。

在调用`start_response`时，就发送了HTTP响应的Header，注意Header只能发送一次，也就是只能调用一次`start_response()`函数。`start_response()`函数接收两个参数，一个是HTTP响应码，一个是一组list表示的`HTTP Header`，每个Header用一个包含两个str的tuple表示。

通常情况下，都应该把Content-Type头发送给浏览器。其他很多常用的HTTP Header也应该发送。

然后，函数的返回值'<h1>Hello, web!</h1>'将作为HTTP响应的Body发送给浏览器。

我们关心的就是如何从`environ`这个dict对象拿到HTTP请求信息，然后构造HTML，通过`start_response()`发送Header，最后返回Body。

而`application()`函数的调用由WSGI服务器来实现。最简单的是Python内置了一个WSGI服务器，这个模块叫`wsgiref`，它是用纯Python编写的WSGI服务器的参考实现。所谓“参考实现”是指该实现完全符合WSGI标准，但是不考虑任何运行效率，仅供开发和测试使用。

### demo
hello.py,WSGI服务器：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return '<h1>Hello, %s!</h1>' % (environ['PATH_INFO'][1:] or 'web') #返回URL的后一部分
```
server.py，负责启动WSGI服务器，加载application()函数：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from wsgiref.simple_server import make_server

from hello import application

httpd = make_server('', 8000, application)
print "Serving HTTP on port 8000..."

httpd.serve_forever()
```
无论多么复杂的Web应用程序，入口都是一个WSGI处理函数。HTTP请求的所有输入信息都可以通过environ获得，HTTP响应的输出都可以通过start_response()加上函数返回值作为Body。

复杂的Web应用程序，光靠一个WSGI函数来处理还是太底层了，我们需要在WSGI之上再抽象出Web框架，进一步简化Web开发。


## 使用Web框架

了解了WSGI框架，我们发现：其实一个Web App，就是写一个WSGI的处理函数，针对每个HTTP请求进行响应。

那如果有一百个网页需要响应呢？













>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)和官方文档的一个学习内容的总结。以便于自己后的学习和整理。
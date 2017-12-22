---
title: javaweb总结（一）基础概念
date: 2017-02-14 22:22:50
tags: 
- javaweb
categories: 总结
---
毕设所需，之前的学习先停一下，先主攻`javaweb`。学习参考：

[孤傲苍狼 博客](http://www.cnblogs.com/xdp-gacl/p/3729033.html)

-------

## 静态web
静态web（如html 页面）：指web页面中供人们浏览的数据始终是不变。其存在以下几个缺点：
1、Web页面中的内容无法动态更新，所有的用户每时每刻看见的内容和最终效果都是一样的。
   为了可以让静态的WEB的显示更加好看，可以加入了JavaScript以完成一些页面上的显示特效，但是这些特效都是在客户端上借助于浏览器展现给用户的，所以在服务器上本身并没有任何的变化。
2、静态WEB无法连接数据库，无法实现和用户的交互。

<!-- more -->

## 动态web
动态web：指web页面中供人们浏览的数据是由程序产生的，不同时间点访问web页面看到的内容各不相同。常用动态web资源开发技术：JSP/Servlet、ASP、PHP等。动态WEB中，程序依然使用客户端和服务端，客户端依然使用浏览器（IE、FireFox等），通过网络(Network)连接到服务器上，使用HTTP协议发起请求（Request），现在的所有请求都先经过一个WEB Server Plugin（服务器插件）来处理，此插件用于区分是请求的是静态资源(*.htm或者是*.htm)还是动态资源。

如果WEB Server Plugin发现客户端请求的是静态资源(*.htm或者是*.htm)，则将请求直接转交给WEB服务器，之后WEB服务器从文件系统中取出内容，发送回客户端浏览器进行解析执行。

如果WEB Server Plugin发现客户端请求的是动态资源（*.jsp、*.asp/*.aspx、*.php），则先将请求转交给WEB Container(WEB容器)，在WEB Container中连接数据库，从数据库中取出数据等一系列操作后动态拼凑页面的展示内容，拼凑页面的展示内容后，把所有的展示内容交给WEB服务器，之后通过WEB服务器将内容发送回客户端浏览器进行解析执行。

### 动态Web的实现方式

动态WEB现在的实现手段非常多，较为常见的有以下几种：

1.Microsoft ASP、ASP.NET

微软公司动态WEB开发是比较早的，而且最早在国内最流行的是ASP。ASP就是在HTML语言之中增加了VB脚本，但是标准的开发应用应该是使用ASP+COM，但是实际情况来看，在开发ASP的时候基本上都在一个页面中写上成百上千的代码，页面代码极其混乱。

ASP本身有开发平台的限制：Windows+IIS+SQL Server/Access，ASP只能运行在Windows操作系统上，ASP现在基本上已经淘汰，现在基本上都是使用ASP.NET进行开发，ASP.NET在性能有了很大的改善，而且开发迅速，但是依然受限于平台。ASP.NET中主要是使用C#语言。

2.PHP

PHP开发速度很快，功能强大，跨平台(平台指的就是运行的操作系统)，而且代码也简单。

3.JAVA Servlet/JSP

这是SUN公司(SUN现在已经被Oracle公司收购)主推的B/S架构的实现语言，是基于JAVA语言发展起来的，因为JAVA语言足够简单，而且很干净。
Servlet/JSP技术的性能也是非常高的，不受平台的限制，各个平台基本上都可以使用。而且在运行中是使用多线程的处理方式，所以性能非常高。

SUN公司最早推出的WEB技术推出的是Servlet程序，Servlet程序本身使用的时候有一些问题，所有的程序是采用JAVA代码+HTML的方式编写的，即，要使用JAVA输出语句，一行一行地输出所有的HTML代码，之后，SUN公司受到了ASP的启发，发展出了JSP(Java Server Page)，JSP某些代码的编写效果与ASP是非常相似的。这样可以很方便地使一些ASP程序员转向JSP的学习，加大市场的竞争力度。

## Tomcat
我们这边使用到的是Tomcat作为我们的WEB服务器。Tomcat是一个实现了JAVA EE标准的最小的WEB服务器，是Apache 软件基金会的Jakarta项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。因为Tomcat 技术先进、性能稳定，而且开源免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。学习JavaWeb开发一般都使用Tomcat服务器，该服务器支持全部JSP以及Servlet规范。

我们在使用中，在其安装位置所在的`conf\server.xml`中可以对服务器进行配置。

### 端口配置
```
<Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
```
将其`port="8080"`可改为任意不冲突端口。

### 虚拟目录的映射方式

在`<Host></Host>`这对标签加上`<Context path="/JavaWebApp" docBase="F:\JavaWebDemoProject" />`即可将在F盘下的`JavaWebDemoProject`这个JavaWeb应用映射到`JavaWebApp`这个虚拟目录上，JavaWebApp这个虚拟目录是由Tomcat服务器管理的，JavaWebApp是一个硬盘上不存在的目录，是我们自己随便写的一个目录，也就是虚拟的一个目录，所以称之为"虚拟目录".

## 浏览器与服务器交互的过程

1. 浏览器发出请求，到host文件查询主机名对应的IP；
2. 如果没找到，浏览器去互联网上的dns服务器查询主机名对应的IP；
3. 浏览器根据查询的IP连上web服务器；
4. 浏览器连接到web服务器后，就使用http协议向服务器发送`get`请求，浏览器会向Web服务器以Stream(流)的形式传输数据，告诉Web服务器要访问服务器里面的哪个Web应用下的Web资源；
5. 之后浏览器开始等待，web服务器从http请求中解析出客户机要访问的主机名；
6. web服务器从http请求中解析出客户机要访问的web应用；
7. web服务器从http请求中解析出客户机要访问的web资源；
8. web服务器从对应路径的web资源数据；
9. web服务器回送数据。
10. 浏览器拿到服务器传输给它的数据之后，就可以把数据展现给用户看了。

## JavaWeb应用的组成结构

开发JavaWeb应用时，不同类型的文件有严格的存放规则，否则不仅可能会使web应用无法访问，还会导致web服务器启动报错。

`WEB-INF`：此文件夹用来存放java类、jar包、web应用的配置文件如`web.xml`，**该目录下的文件外界无法非法直接访问，由web服务器负责调用。**
而与此文件夹位于同一文件夹内的文件可以直接访问。
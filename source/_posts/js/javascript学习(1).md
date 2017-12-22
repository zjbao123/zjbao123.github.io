---
title: JavaScript学习(1)基础概念
date: 2016-08-22 22:14:50
tags: 
- JavaScript
categories: 总结
---

### 什么是JavaScript？

JavaScript是一种运行在浏览器中的解释型的编程语言。你在电脑、手机、平板上浏览的所有的网页，以及无数基于HTML5的手机App，交互逻辑都是由JavaScript驱动的。
<!-- more -->
### 为什么我们要学JavaScript？

因为只有JavaScript能跨平台、跨浏览器驱动网页，与用户交互。。相反，随着HTML5在PC和移动端越来越流行，JavaScript变得更加重要了。并且，新兴的Node.js把JavaScript引入到了服务器端，JavaScript变得非常全能

### JavaScript与Java的关系

当时Java语言非常红火，所以网景公司希望借Java的名气来推广，但事实上JavaScript除了语法上有点像Java，其他部分基本上没啥关系。

### ECMAScript

ECMAScript是为了让JavaScript成为全球标准，几个公司联合ECMA（European Computer Manufacturers Association）组织定制了JavaScript语言的标准，被称为ECMAScript(简称ES)标准。大多数时候与JavaScript相同。因为JavaScript已经被网景注册。

###　JavaScript入门

JavaScript嵌入网页：

1.通常我们都把JavaScript代码放到`<head>`中。用`<script>...</script>`包含
2.把JavaScript代码放到一个单独的.js文件，然后在HTML中通过`<script src="..."></script>`引入这个文件,这样有利于维护。在页面中存在多行，则按顺序执行。

`<script>`有个`type="text/javascript"`属性，默认如此，不写也罢。

### 编写JavaScript的工具

1.Sublime Text
2.Notepad++
3.atom

均是免费的，推荐Sublime Text，插件丰富，个性化自己的编辑器，不过时而会跳出购买界面，不购买也可以继续使用，不过支持一下也好。

### 如何调试JavaScript

首先安装`chrome`,同样也是插件丰富，可以完美个性化。

安装后，按`F12`进入开发者模式，点击`控制台(Console)`,可以直接执行JavaScript代码。
也可在点击`源码(Sources)`,进行断点，单步调试。

**请注意，JavaScript严格区分大小写，如果弄错了大小写，程序将报错或者运行不正常。**



----
学习参考：
[廖雪峰的JavaScript教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000)
[JavaScript 标准参考教程 阮一峰](http://javascript.ruanyifeng.com/)
[李炎恢的JavaScript视频教程](http://www.ycku.com/javascript/?utm_source=caibaojian.com)
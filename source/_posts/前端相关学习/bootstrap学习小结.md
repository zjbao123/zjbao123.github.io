---
title: bootstrap学习小结
date: 2016-07-20 20:22:50
tags: 
- html
- bootstrap
categories: 总结
---

## bootstrap简介

### 什么是bootstrap？

2011年，twitter的“一小撮”工程师为了提高他们内部的分析和管理能力，用业余时间为他们的产品构建了一套易用、优雅、灵活、可扩展的前端工具集--BootStrap。在github上开源之后，迅速成为该站上最多人watch&fork的项目。大量工程师踊跃为该项目贡献代码，社区惊人地活跃，代码版本进化非常快速，官方文档质量极其高(可以说是优雅)，同时涌现了许多基于Bootstrap建设的网站：界面清新、简洁;要素排版利落大方。<!-- more -->

GitHub上这样介绍 bootstrap：

 * 简单灵活可用于架构流行的用户界面和交互接口的html、css、javascript工具集。
 *  基于html5、css3的bootstrap，具有大量的诱人特性：友好的学习曲线，卓越的兼容性，响应式设计，12列格网，样式向导文档。
 *  自定义JQuery插件，完整的类库，基于Less等。


### Bootstrap样式

Bootstrap框架在这一部分做了一定的变更，不再一味追求归零，而是更注重重置可能产生问题的样式（如，body,form的margin等），保留和坚持部分浏览器的基础样式，解决部分潜在的问题，提升一些细节的体验，具体说明如下：

* 移除body的margin声明
* 设置body的背景色为白色
* 为排版设置了基本的字体、字号和行高
* 设置全局链接颜色，且当链接处于悬浮“:hover”状态时才会显示下划线样式




### 强调

`b`,`strong`  让文本直接加粗

`.lead`  让一个段落p突出显示

`em`,`i` 还可以通过使用标签或将元素设置样式font-style值为italic实现斜体

* .text-muted：提示，使用浅灰色（#999）

* .text-primary：主要，使用蓝色（#428bca）

* .text-success：成功，使用浅绿色(#3c763d)

* .text-info：通知信息，使用浅蓝色（#31708f）

* .text-warning：警告，使用黄色（#8a6d3b）

* .text-danger：危险，使用褐色（#a94442）


### 对齐风格


*  .text-left：左对齐

*  .text-center：居中对齐

*  .text-right：右对齐

*  .text-justify：两端对齐

### 列表

.list-unstyled  给无序列表添去除默认的列表样式的风格。

.dl-horizontal  给定义列表设置为水平（即将dt设置为浮动）

### 代码风格

1、`code`：一般是针对于单个单词或单个句子的代码，淡粉底
2、`pre`：一般是针对于多行代码（也就是成块的代码），灰底
3、`kbd`:一般是表示用户要通过键盘输入的内容,黑底

在pre标签上添加类名“.pre-scrollable”，就可以控制代码块区域最大高度为340px，一旦超出这个高度，就会在Y轴出现滚动条。

### 表格

Bootstrap为表格提供了1种基础样式和4种附加样式以及1个支持响应式的表格。
1. `.table`：基础表格

 1. 给表格设置了margin-bottom:20px以及设置单元内距

 2. 在thead底部设置了一个2px的浅灰实线

 3. 每个单元格顶部设置了一个1px的浅灰实线


2. .table-striped：斑马线表格(隔行有一个浅灰色的背景色)

3. .table-bordered：带边框的表格(中规中矩)

4. .table-hover：鼠标悬停高亮的表格(当你鼠标悬浮在某一单元格上时，单元格所在行的背景色都会变成浅灰色。)

5. .table-condensed：紧凑型表格(变挤而已)

6. .table-responsive：响应式表格就是当屏幕小于768px时，表格下方会出现滚轮效果等）

表格行的对应类：

![表格图](http://7xsp7y.com1.z0.glb.clouddn.com/53ad213f0001b08807340508.jpg)

### 表单

表单中常见的元素主要包括：文本输入框、下拉选择框、单选按钮、复选按钮、文本域和按钮等。

表单内容用for和id进行元素绑定

`form-horizontal`　水平表单，标签居左，表单控件居右

`sr-only`　将标签隐藏

为了让控件在各种表单风格中样式不出错，需要添加类名“form-control”

控件`input`必须添加type类型来定义样式

控件`select`增加multiple多行显示

`textarea`设置rows可定义其高度，在`from-control`中无序设置cols(宽度)属性

`input-sm`让控件比正常大小更小

`input-lg`让控件比正常大小更大



### 图像和图标

#### 图像

可通过`src="http://placehold.it/140x140"`获取图像样式

1. img-responsive：响应式图片，主要针对于响应式设计

2. img-rounded：圆角图片

3. img-circle：圆形图片

4. img-thumbnail：缩略图片

#### 图标

通过添加类，如搜索图标，即`glyphicon glyphicon-search`


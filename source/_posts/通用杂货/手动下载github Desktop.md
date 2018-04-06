---
title: 手动下载github Desktop
date: 2016-11-29 00:34:50
tags: 
- github
categories: 总结
---

`github Desktop`是`github`推出的桌面版图形界面，由于最近我升了win10，就需要重新下载`github Desktop`，又要经历一遍痛苦的过程。为什么说是痛苦的呢？安装的时候一直卡在下载界面，下了几十兆就断开连接，又要重新开始。挂vpn，加host，加信任站点，能试的都试了一遍，还是不行。关于这个[问题](https://www.zhihu.com/question/23110947),轮子哥说的好，“我也遇到过这种问题，后来我解决了他，方法就是：明天再说。”看到知乎里有大神自己手动下载下来了github离线版，可是就是不甘心啊，想要自己来试试这个过程，也顺便做个记录。
<!-- more -->
通过一些搜寻，还是找到一些方法的。可以先下载`http://github-windows.s3.amazonaws.com/GitHub.application`(直接在网页打开并下载)，接下来运行`GitHub.application`，会发现报错了。原因是没有找到`GitHub.exe.manifest`，继续找了对应的url，也进行了下载并放在了其目录下。一开始并不知道`GitHub.exe.manifest`的作用，还是傻瓜式的报错下文件，但参考了[姚泽源](http://www.yaozeyuan.online/2015/10/02/2015%E5%B9%B410%E6%9C%882%E6%97%A5081458-github-windows%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85%E7%89%88/)所说的内容之后，我开始查看其源码。

发现其`file`格式里的`name`和`codebase`里的内容即为文件名，于是通过正则`codebase="(.*?)"`匹配得到了所有的文件名。
![正则匹配文件名](http://7xsp7y.com1.z0.glb.clouddn.com/blog/20161128/234440549.png)

然后，将其匹配出来的复制到另一个文本中。图中仅仅是`codebase`的部分内容。
![codebase的部分内容](http://7xsp7y.com1.z0.glb.clouddn.com/blog/20161129/002029641.png)

再使用正则将其改为URL格式，再用迅雷进行下载。
![mark](http://7xsp7y.com1.z0.glb.clouddn.com/blog/20161129/002153759.png)
其中会出现有部分文件无法下载，可用浏览器下载并放在对应目录下。

以下是我[最新版github离线版安装包](http://pan.baidu.com/s/1i59bG1n)可以直接下载。
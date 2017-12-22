---
title: 优化你的 nexT
date: 2016-04-07 13:07:57
tags:
- hexo
- nexT优化
categories: 优化
---


![next](http://7xsp7y.com2.z0.glb.qiniucdn.com/16-4-7/64957320.jpg)
## 什么是NexT？
[NexT](https://github.com/iissnan/hexo-theme-next)为hexo一个较为经典的主题，我的博客目前所用的就是这个主题。感谢作者[iissnan](https://github.com/iissnan)的提供。<!-- more -->


`NexT` 主旨在于简洁优雅且易于使用，所以首先要尽量确保 `NexT` 的简洁易用性。在保证其原有的基础上，我对其一些基本功能进行了优化。
如果你也想尝试与我一样的主题，可以参考[NexT帮助文档](http://theme-next.iissnan.com/getting-started.html)来个性化你的`NexT`。或者你也可以选择一个Hexo主题：
>选择主题建议遵循KISS原则,即简单就是美。

* Hexo Themes - http://hexo.io/themes/
* Jacman - http://wsgzao.github.io/post/hexo-jacman/

## 如何设置阅读全文
`hexo`提供了在文章中使用 <!-- more --> 手动进行截断的方式，前提是需要在主题文件夹下的`_config.yml`设置：
>scroll\_to\_more: true

## hexo NexT主题首页title的优化
更改`index.swig`文件，文件路径为`your-hexo-site\themes\next\layout`，将下面代码

```
	{% block title %} {{ config.title }} {% endblock %}
```

更改为

```
	{% block title %} {{ config.title }} - {{ theme.description }} {% endblock %}
```

即可实现使你的首页显示`网站名称 - 网站描述`这种格式了。

## 设置最近访客菜单
本设置参考学习了`arao`的博客[动动手指，给你的Hexo站点添加最近访客（多说篇）](http://www.arao.me/2015/hexo-next-theme-optimize-duoshuo/)，并且将多说评论UA与其相结合，实现了最近访客功能以及评论栏的背景及评论栏头像360度旋转。
## 第三方服务配置
第一次接触的`NexT`的时候，按照教程来还是遇到了挺多的问题。在集成第三方服务的时候，由于`NexT`默认使用国外比较流行的disqus，为了与国内相匹配，我将其修改为“多说”评论系统。

在每一条多说的评论之后我想要显示出评论者所使用的代理消息（如 操作系统、浏览器），可是按照教程之后发现并没有出现。![](http://theme-next.iissnan.com/uploads/duoshuo-ua.png)

后参考了HelloDog的博客[多说自定义CSS头像和多说评论显示UA](http://wsgzao.github.io/post/duoshuo/)才明白需要到多说后台自定义CSS（此CSS样式包括了最近访客功能以及评论栏的背景及评论栏头像360度旋转的功能），具体步骤为：

>注册/登录多说后台->设置->基本设置->自定义CSS



```
	/*UA Start*/
	span.this_ua {
	    background-color: #ccc!important;
	    border-radius: 4px;
	    padding: 0 5px!important;
	    margin: 0 1px!important;
	    border: 1px solid #BBB!important;
	    color: #fff;
	    /*text-transform: Capitalize!important;
	    float: right!important;
	    line-height: 18px!important;*/
	}
	.this_ua.platform.Windows{
	    background-color: #39b3d7!important;
	    border-color: #46b8da!important;
	}
	.this_ua.platform.Linux {
	    background-color: #3A3A3A!important;
	    border-color: #1F1F1F!important;
	}
	.this_ua.platform.Ubuntu {
	    background-color: #DD4814!important;
	    border-color: #DD4814!important;
	}
	.this_ua.platform.Mac {
	    background-color: #666666!important;
	    border-color: #666666!important;
	}
	.this_ua.platform.Android {
	    background-color: #98C13D!important;
	    border-color: #98C13D!important;
	}
	.this_ua.platform.iOS {
	    background-color: #666666!important;
	    border-color: #666666!important;
	}
	.this_ua.browser.Chrome{
	    background-color: #EE6252!important;
	    border-color: #EE6252!important;
	}
	.this_ua.browser.Chromium{
	    background-color: #EE6252!important;
	    border-color: #EE6252!important;
	}
	.this_ua.browser.Firefox{
	    background-color: #f0ad4e!important;
	    border-color: #eea236!important;
	}
	.this_ua.browser.IE{
	    background-color: #428bca!important;
	    border-color: #357ebd!important;
	}
	.this_ua.browser.Edge{
	    background-color: #428bca!important;
	    border-color: #357ebd!important;
	}
	.this_ua.browser.Opera{
	    background-color: #d9534f!important;
	    border-color: #d43f3a!important;
	}
	.this_ua.browser.Maxthon{
	    background-color: #7373B9!important;
	    border-color: #7373B9!important;
	}
	.this_ua.browser.Safari{
	    background-color: #666666!important;
	    border-color: #666666!important;
	}
	.this_ua.sskadmin {
	    background-color: #00a67c!important;
	    border-color: #00a67c!important;
	}
	/*UA End*/
	/*Head Start*/
	#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current {
	    border: 0px;
	    color: #6D6D6B;
	    text-shadow: none;
	    background: #F3F3F3;
	}
	
	#ds-thread #ds-reset .ds-highlight {
	    font-family: Microsoft YaHei, "Helvetica Neue", Helvetica, Arial, Sans-serif;
	    ;font-size: 100%;
	    color: #6D6D6B !important;
	}
	
	#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current:hover {
	    color: #696a52;
	    background: #F2F2F2;
	}
	
	#ds-thread #ds-reset a.ds-highlight:hover {
	    color: #696a52 !important;
	}
	
	#ds-thread {
	    padding-left: 15px;
	}
	
	#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset #ds-hot-posts {
	    overflow: visible;
	}
	
	#ds-thread #ds-reset .ds-post-self {
	    padding: 10px 0 10px 10px;
	}
	
	#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset .ds-post-self {
	    border: 0 !important;
	}
	
	#ds-reset .ds-avatar, #ds-thread #ds-reset ul.ds-children .ds-avatar {
	    top: 15px;
	    left: -20px;
	    padding: 5px;
	    width: 36px;
	    height: 36px;
	    box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
	    border-radius: 46px;
	    background: #FAFAFA;
	}
	
	#ds-thread .ds-avatar a {
	    display: inline-block;
	    padding: 1px;
	    width: 32px;
	    height: 32px;
	    border: 1px solid #b9baa6;
	    border-radius: 50%;
	    background-color: #fff !important;
	}
	
	#ds-thread .ds-avatar a:hover {
	}
	
	#ds-thread .ds-avatar > img {
	    margin: 2px 0 0 2px;
	}
	
	#ds-thread #ds-reset .ds-replybox {
	    box-shadow: none;
	}
	
	#ds-thread #ds-reset ul.ds-children .ds-replybox.ds-inline-replybox a.ds-avatar,
	#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar {
	    left: 0;
	    top: 0;
	    padding: 0;
	    width: 32px !important;
	    height: 32px !important;
	    background: none;
	    box-shadow: none;
	}
	
	#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar img {
	    width: 32px !important;
	    height: 32px !important;
	    border-radius: 50%;
	}
	
	#ds-reset .ds-replybox a.ds-avatar,
	#ds-reset .ds-replybox .ds-avatar img {
	    padding: 0;
	    width: 32px !important;
	    height: 32px !important;
	    border-radius: 5px;
	}
	
	#ds-reset .ds-avatar img {
	    width: 32px !important;
	    height: 32px !important;
	    border-radius: 32px;
	    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.22);
	    -webkit-transition: .8s all ease-in-out;
	    -moz-transition: .4s all ease-in-out;
	    -o-transition: .4s all ease-in-out;
	    -ms-transition: .4s all ease-in-out;
	    transition: .4s all ease-in-out;
	}
	
	.ds-post-self:hover .ds-avatar img {
	    -webkit-transform: rotateX(360deg);
	    -moz-transform: rotate(360deg);
	    -o-transform: rotate(360deg);
	    -ms-transform: rotate(360deg);
	    transform: rotate(360deg);
	}
	
	#ds-thread #ds-reset .ds-comment-body {
	    -webkit-transition-delay: initial;
	    -webkit-transition-duration: 0.4s;
	    -webkit-transition-property: all;
	    -webkit-transition-timing-function: initial;
	    background: #F7F7F7;
	    padding: 15px 15px 15px 47px;
	    border-radius: 5px;
	    box-shadow: #B8B9B9 0 1px 3px;
	    border: white 1px solid;
	}
	
	#ds-thread #ds-reset ul.ds-children .ds-comment-body {
	    padding-left: 15px;
	}
	
	#ds-thread #ds-reset .ds-comment-body p {
	    color: #787968;
	}
	
	#ds-thread #ds-reset .ds-comments {
	    border-bottom: 0px;
	}
	
	#ds-thread #ds-reset .ds-powered-by {
	    display: none;
	}
	
	#ds-thread #ds-reset .ds-comments a.ds-user-name {
	    font-weight: normal;
	    color: #3D3D3D !important;
	}
	
	#ds-thread #ds-reset .ds-comments a.ds-user-name:hover {
	    color: #D32 !important;
	}
	
	#ds-thread #ds-reset #ds-bubble {
	    display: none !important;
	}
	
	#ds-thread #ds-reset #ds-hot-posts {
	    border: 0;
	}
	
	#ds-thread #ds-reset .ds-textarea-wrapper textarea {
		background: url(http://ww4.sinaimg.cn/small/649a4735gw1et7gnhy5fej20zk0m8q3q.jpg) right no-repeat;
	}

	#ds-recent-visitors .ds-avatar {
		float: left
	}
	/*隐藏多说底部版权*/
	#ds-thread #ds-reset .ds-powered-by {
		display: none;
	}
	
	#ds-thread #ds-reset .ds-comment-body:hover {
	    background-color: #F1F1F1;
	    -webkit-transition-delay: initial;
	    -webkit-transition-duration: 0.4s;
	    -webkit-transition-property: all;
	    -webkit-transition-timing-function: initial;
	}
	
```

添加上述代码，最后在命令行中 `hexo cl`&`hexo g`&`hexo s`,最后显示如下图所示：![评论](http://7xsp7y.com2.z0.glb.qiniucdn.com/16-4-7/11047875.jpg)实现了上述功能。

## 添加`小心！`

再次感谢开源

效果很赞，整个页面渐进式摇摆，摇摆，还有音乐

在 Hexo\themes\next\layout_partials\header.swig 中的 ul 标签加入如下 li 代码：
```
<li> <a title="把这个链接拖到你的Chrome收藏夹工具栏中" href='javascript:(function() {
    function c() {
        var e = document.createElement("link");
        e.setAttribute("type", "text/css");
        e.setAttribute("rel", "stylesheet");
        e.setAttribute("href", f);
        e.setAttribute("class", l);
        document.body.appendChild(e)
    }
 
    function h() {
        var e = document.getElementsByClassName(l);
        for (var t = 0; t < e.length; t++) {
            document.body.removeChild(e[t])
        }
    }
 
    function p() {
        var e = document.createElement("div");
        e.setAttribute("class", a);
        document.body.appendChild(e);
        setTimeout(function() {
            document.body.removeChild(e)
        }, 100)
    }
 
    function d(e) {
        return {
            height : e.offsetHeight,
            width : e.offsetWidth
        }
    }
 
    function v(i) {
        var s = d(i);
        return s.height > e && s.height < n && s.width > t && s.width < r
    }
 
    function m(e) {
        var t = e;
        var n = 0;
        while (!!t) {
            n += t.offsetTop;
            t = t.offsetParent
        }
        return n
    }
 
    function g() {
        var e = document.documentElement;
        if (!!window.innerWidth) {
            return window.innerHeight
        } else if (e && !isNaN(e.clientHeight)) {
            return e.clientHeight
        }
        return 0
    }
 
    function y() {
        if (window.pageYOffset) {
            return window.pageYOffset
        }
        return Math.max(document.documentElement.scrollTop, document.body.scrollTop)
    }
 
    function E(e) {
        var t = m(e);
        return t >= w && t <= b + w
    }
 
    function S() {
        var e = document.createElement("audio");
        e.setAttribute("class", l);
        e.src = i;
        e.loop = false;
        e.addEventListener("canplay", function() {
            setTimeout(function() {
                x(k)
            }, 500);
            setTimeout(function() {
                N();
                p();
                for (var e = 0; e < O.length; e++) {
                    T(O[e])
                }
            }, 15500)
        }, true);
        e.addEventListener("ended", function() {
            N();
            h()
        }, true);
        e.innerHTML = " <p>If you are reading this, it is because your browser does not support the audio element. We recommend that you get a new browser.</p> <p>";
        document.body.appendChild(e);
        e.play()
    }
 
    function x(e) {
        e.className += " " + s + " " + o
    }
 
    function T(e) {
        e.className += " " + s + " " + u[Math.floor(Math.random() * u.length)]
    }
 
    function N() {
        var e = document.getElementsByClassName(s);
        var t = new RegExp("\\b" + s + "\\b");
        for (var n = 0; n < e.length; ) {
            e[n].className = e[n].className.replace(t, "")
        }
    }
 
    var e = 30;
    var t = 30;
    var n = 350;
    var r = 350;
    var i = "//s3.amazonaws.com/moovweb-marketing/playground/harlem-shake.mp3";
    var s = "mw-harlem_shake_me";
    var o = "im_first";
    var u = ["im_drunk", "im_baked", "im_trippin", "im_blown"];
    var a = "mw-strobe_light";
    var f = "//s3.amazonaws.com/moovweb-marketing/playground/harlem-shake-style.css";
    var l = "mw_added_css";
    var b = g();
    var w = y();
    var C = document.getElementsByTagName("*");
    var k = null;
    for (var L = 0; L < C.length; L++) {
        var A = C[L];
        if (v(A)) {
            if (E(A)) {
                k = A;
                break
            }
        }
    }
    if (A === null) {
        console.warn("Could not find a node of the right size. Please try a different page.");
        return
    }
    c();
    S();
    var O = [];
    for (var L = 0; L < C.length; L++) {
        var A = C[L];
        if (v(A)) {
            O.push(A)
        }
    }
})()    '>小心！</a> </li>
```
很刺激的特效 = =

总体感觉，nexT的主题很有特色，简单，有极客范。
---
title: 【python】用python自动发微博
date: 2016-10-23 17:14:50
tags: 
- python
- 趣玩
categories: 总结
---

首先声明一下，这只是一个小demo，用来作为我学习python的练习，一个除了学（zhuang）习（bi）并无大用的小脚本而已。

咳咳，下面开始手把手教学，大家好好听啊【敲黑板】
<!--more-->
## 第一步：你应该要有自己的`app key `和 `app scret`
[移动API申请](http://open.weibo.com/development/mobile)

点击上面这个链接就可以进入开发者中心申请`app key `和 `app scret`啦，当然前提是你要完善一下个人信息，只要基本信息完善一下即可。然后激活邮箱，你申请一个就可以得到上面的所需信息啦。

之后，在高级设置中将回调函数修改一下：`https://api.weibo.com/oauth2/default.html` ，这个需要和之后的python code里面的 `callback url`一致！！！

## 第二步：安装所需插件
这里要感谢一下`廖雪峰`大牛提供的SDK：`sinaweibopy`
```
pip install sinaweibopy
pip install PIL
```
确认一下你的python版本，这里用的是`python 2.7`，会用2.7，3自然不是什么难事。

## python Code

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'


from weibo import APIClient
import weather

def get_access_token(app_key, app_secret, callback_url):
    client = APIClient(app_key=app_key, app_secret=app_secret, redirect_uri=callback_url)
    # 获取授权页面网址
    auth_url = client.get_authorize_url()
    print auth_url

    # 在浏览器中访问这个URL，会跳转到回调地址，回调地址后面跟着code，输入code
    code = raw_input("Input code:")
    r = client.request_access_token(code)
    access_token = r.access_token
    # token过期的UNIX时间
    expires_in = r.expires_in
    print 'access_token:', access_token
    print 'expires_in:', expires_in

    return access_token, expires_in
def init_login():
    app_key = 'xxxxxxx'
    app_secret = 'xxxxxxxxxx'
    callback_url = 'https://api.weibo.com/oauth2/default.html'

    access_token, expires_in = get_access_token(app_key, app_secret, callback_url)
    # 上面的语句运行一次后，可保存得到的access token，不必每次都申请
    #print "access_token = %s, expires_in = %s" % (access_token, expires_in)
    # access_token = 'xxxxxxxx'
    # expires_in = 'xxxxxx'

    client = APIClient(app_key=app_key, app_secret=app_secret, redirect_uri=callback_url)
    client.set_access_token(access_token, expires_in)
    return client


def send_pic(client,picpath,message):
    # send a weibo with img
    f = open(picpath, 'rb')
    mes = message.decode('utf-8')
    client.statuses.upload.post(status=mes, pic=f)
    f.close()  # APIClient不会自动关闭文件，需要手动关闭
    print u"发送成功！"

def send_mes(client,message):
    utext = unicode(message,"UTF-8")
    client.post.statuses__update(status=utext)
    print u"发送成功！"


if __name__ == '__main__':
    client = init_login()
    weather.draw_pic(weather.weather())
    mes = "鲍先森又被盗号啦哈哈哈，我特意来跟大家汇报杭州今日天气："
    send_pic(client,'2.jpg',mes)

```

看完源码会发现，这个`weather`是什么东西？我看这个发微博单调无比，给他新增了一个功能，根据每日天气预报，绘制一张天气预报的图。
![天气预报](http://i1.piimg.com/567571/437f626213a4472e.jpg)

weather.py:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
__author__ = 'zjbao123'
import json
import urllib2
from PIL import Image,ImageDraw,ImageFont

def weather():
    # 获取每日天气数据
    try:
        url = 'http://api.map.baidu.com/telematics/v3/weather?location=%E6%9D%AD%E5%B7%9E&output=json&ak=KPGX6sBfBZvz8NlDN5mXDNBF&callback='
        s=json.loads(urllib2.urlopen(url).read())
        s1 = s["results"][0]["weather_data"][0]["temperature"]
        s2 = s["results"][0]["weather_data"][0]["weather"]
        # print s["results"][0]["currentCity"]
        # print s["results"][0]["weather_data"][0]["temperature"]
        # print s["results"][0]["weather_data"][0]["weather"]
        return s1,s2
    except :
        print"error"
def draw_pic(l):
    img = Image.open('test.jpg')
    draw = ImageDraw.Draw(img)
    myfont = ImageFont.truetype(u'C:/windows/fonts/逼格锐线体简4.0 (2).TTF', size=50) #字体自己改
    draw.text((img.size[0]/6,img.size[1]/5),unicode(l[0]),font=myfont, fill = (0,177,106))
    draw.text((img.size[0]/3,img.size[1]/5+150),unicode(l[1]),font=myfont, fill = (0,128,131))
    img.save('2.jpg','jpeg')
    print 'ok'
```
结果如图示：
![微博截图](http://i1.piimg.com/567571/6ab683a65bd8d90d.jpg)

最后贴上我的github链接吧：[repo](https://github.com/zjbao123/python_sinaWeiBo)


-----
参考：
http://www.guokr.com/post/475564/
https://www.zhihu.com/question/36960036
---
title: 同步博客coding和github并实现本地上传图片至博客
date: 2017-10-18 18:01:50
tags: 
- 博客优化
categories: 总结
---

## 同步博客至coding和github

由于有些时候github page 会被墙，需要翻墙才能浏览。所以找一个国内的博客也变得十分需要。在查找之后，发现[coding.net](http://coding.net)非常的不错。
![coding](http://i2.buimg.com/567571/1f5ff62372ee57a0.png)
查了些资料，成功实现在国内托管平台coding同步自己的博客。
<!-- more -->
### 一.创建秘钥

1.安装`git`

2.安装完成后运行`git shell`

3.输入`cd ~/.ssh` ,跳转到你的.ssh目录

4.将目录下生成的 `id_rsa.pub` 复制出来部署至`coding` 和 `github`.

### 二.部署秘钥
1.在coding中新建项目，项目名与Global Key，即用户名。

2.在github和coding中部署ssh相同的密钥，添加后，在Git命令窗口输入如下命令进行测试：

```
ssh -T git@git.coding.net
ssh -T git@github.com
```
出现下列提示则说明添加成功：
```
Coding.net Tips : [Hello ! You've conected to Coding.net by SSH successfully! ]
```

3.将博客根目录下的 `_config.yml` 的一致deploy修改成类似代码

```
deploy:
  type: git
  repo:
      github: git@github.com:username/username.github.io.git,master
      coding: git@git.coding.net:username/username.git,master
```

4.之后使用部署命令来同步博客到coding和github
```
hexo cl
hexo g
hexo d
```

5.那直接访问 username.coding.me就能访问博客

## 本地上传图片至博客

之前在看博客的时候看到一个开源库觉得不错，可以实现使用本地图片。

地址如下：[hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)

1.修改配置文件_config.yml
```
post_asset_folder:true

```

2.在hexo的目录下执行
```
npm install hexo-asset-image --save
```

3.完成安装后用hexo新建文章的时候会发现_posts目录下面会多出一个和文章名字一样的文件夹。图片就可以放在文件夹下面。

通过上述例子进行访问。

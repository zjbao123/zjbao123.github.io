---
title: Mysql数据导出至文件
date: 2017-12-01 11:22:50
tags: 
- 数据库
categories: 总结
---

在项目中需要将一些简单的sql语句导出至文件,顺便也把其他的一些操作导出做一个记录.

<!--more-->

```
#sql语句导出
mysql -P(port) -h(host) -u(user) -p(password) -D(DBname) -N <(需要导入的查询脚本文件) >(需要导出的生成脚本文件)
# 数据库内容导出
mysqldump -P(port) -h(host) -u(user) -p(password) -D(DBname) >(需要导出的生成脚本文件)
# 数据库内容导出
mysqldump -P(port) -h(host) -u(user) -p(password) -D(DBname) test(表名) >(需要导出的生成脚本文件)
# 跨机备份数据库
mysqldump --host=host1 --opt sourceDDBname | mysql --host=host2 -C targetDb
#只备份表结构
mysqldump --no-data --databases mydatabase1 mydatabase2 mydatabase3 > test.dump
```

1. `-N`: 不展示第一行的列名
2.`>`: 输出文件
3.`>`: 输入文件,可用于还原备份,导入sql语句
4. --opt: 如果加上--opt参数则生成的dump文件中稍有不同：

		. 建表语句包含drop table if exists tableName
		. insert之前包含一个锁表语句lock tables tableName write，insert之后包含unlock tables
5.-C: 指示主机间的数据传输使用数据压缩
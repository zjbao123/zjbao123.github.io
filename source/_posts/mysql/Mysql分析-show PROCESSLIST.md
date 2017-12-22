---
title: Mysql分析-show PROCESSLIST
date: 2017-12-12 16:22:50
tags: 
- mysql
categories: 总结
---

mysql的`show PROCESSLIST`可以在你的mysql性能出现问题的时候,来查看显示的目前的进程。

<!--more-->

## 各列的含义

1.id：一个标识，你要kill一个语句的时候使用，例如 mysql> kill 207;
2.user：显示当前用户，如果不是root，这个命令就只显示你权限范围内的sql语句;
3.host：显示这个语句是从哪个ip 的哪个端口上发出的，可用来追踪出问题语句的用户;
4.db：显示这个进程目前连接的是哪个;
5.command：显示当前连接的执行的命令，一般就是休眠（sleep），查询（query），连接（connect）;
6.time：此这个状态持续的时间，单位是秒;
7.state：显示使用当前连接的sql语句的状态，很重要的列，state只是语句执行中的某一个状态，例如查询，需要经过copying
 to tmp table，Sorting result，Sending data等状态才可以完成;
8.info：显示这个sql语句，因为长度有限，所以长的sql语句就显示不全，但是一个判断问题语句的重要依据

## state名词含义

### Sleep

通常代表资源未释放，如果是通过连接池，sleep状态应该恒定在一定数量范围内，例如：
数据查询时间为0.1秒，而网络输出需要1秒左右，原本数据连接在0.1秒即可释放，但是因为前端程序未执行close操作，直接输出结果，那么在结果未展现在用户桌面前，该数据库连接一直维持在sleep状态

### Locked

操作被锁定，通常使用innodb可以很好的减少locked状态的产生

### Copy to tmp table

索引及现有结构无法涵盖查询条件时，会建立一个临时表来满足查询要求，产生巨大的i/o压力Copy to tmp table通常与连表查询有关，建议减少关联查询或者深入优化查询语句，如果出现此状态的语句执行时间过长，会严重影响其他操作，此时可以kill掉该操作

### Sending data

Sending data并不是发送数据，是从物理磁盘获取数据的进程，如果你的影响结果集较多，那么就需要从不同的磁盘碎片去抽取数据，如果sending
 data连接过多，通常是某查询的影响结果集过大，也就是查询的索引项不够优化

### Storing result to query cache

如果频繁出现此状态，使用set profiling分析，如果存在资源开销在SQL整体开销的比例过大（即便是非常小的开销，看比例），则说明query
 cache碎片较多，使用flush query cache可即时清理，Query cache参数可适当酌情设置.

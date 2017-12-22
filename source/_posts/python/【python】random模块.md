---
title: 【python】random模块小结
date: 2016-10-16 13:14:50
tags: 
- python
categories: 总结
---
尽管`help（random）`一下就能出来模块内各个函数的用法，不过我也想深入一步看一下`random`的具体用法。
<!-- more -->
`random`是用于生成随机数的，我们可以利用它随机生成数字或者选择字符串。

不过真的是一个很厉害的模块，里面有随机正太分布，帕累托分布，高斯分布，β分布，γ分布，三角分布，威布尔分布等等各种函数，这些就不细讲了，用到再去看就行，`门在哪知道就好`，不过我这边列出一些比较常用的。

* `random.random()` 

用来随机生成一个0到1之间的浮点数，包括零。

* `randint(a, b)`

用来生成[a,b]之间的随意整数，包括两个边界值。

* `random.uniform(a,b)`
用来生成[a,b]之间的随意浮点数，包括两个边界值。

* `choice(seq)` 
从一个非空序列选出随机一个元素。seq泛指list，tuple，字符串等

* `randrange(start, stop[, step = 1])`
这个就是random和range函数的合二为一了。但注意，range用法有变。


以上是一些常规想得到的用法。一下介绍一些不是很想得到的用法。

* `random.shuffle(x[,random]) `
正如函数名所表示的意思，`shuffle`，洗牌，将一个列表中的元素打乱。

* `random.sample(sequence,k) `

sample，样品，从有序列表中选k个作为一个片段返回。


* `random.seed ( [x] )`
`x`:改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。

一般计算机的随机数都是伪随机数，以一个真随机数（种子）作为初始条件，然后用一定的算法不停迭代产生随机数。


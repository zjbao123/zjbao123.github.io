---
title: python爬虫入门(3)
date: 2016-10-17 21:10:02
tags:
- python
- 爬虫
categories: 总结
---

## 正则表达式

这个在之前的博客有讲过，今天就当是复习吧。

正则表达式可以把前面获得到的页面提取整理出来。

>它是对字符串操作的一种逻辑公式，就是用事先定义好的一些特定字符、及这些特定字符的组合，组成一个“规则字符串”，这个“规则字符串”用来表达对字符串的一种过滤逻辑。

<!-- more -->

正则表达式的大致匹配过程是：
1.依次拿出表达式和文本中的字符比较，
2.如果每一个字符都能匹配，则匹配成功；一旦有匹配不成功的字符则匹配失败。
3.如果表达式中有量词或边界，这个过程会稍微有一些不同。

正则表达式的匹配规则：
![匹配规则](http://images.cnblogs.com/cnblogs_com/huxi/Windows-Live-Writer/Python_10A67/pyre_ebb9ce1c-e5e8-4219-a8ae-7ee620d5f9f1.png)

### re模块
Python 自带了re模块，它提供了对正则表达式的支持。主要用到的方法列举如下：

* `compile(pattern, flags=0)`

 将正则表达式字符串形式编译为一个Pattern实例。
 ```
 pattern = re.compile(r'hello')
 ```

 字符串前加r是为了防止字符串转义。 在参数中我们传入了原生字符串对象，通过compile方法编译生成一个pattern对象，然后我们利用这个对象来进行进一步的匹配。

* `escape(pattern)`

 避开所有非字母数字的字符模式，即虽然含有特殊字符，不需要用反斜杠注释。

* `findall(pattern, string, flags=0)`

 返回一个非重叠的字符串匹配列表。
 列表中每个元素的值的类型，取决于你的正则表达式的写法:
	* 元组tuple：当你的正则表达式中有（带捕获的）分组（简单可理解为有括号）而tuple的值，是各个group的值所组合出来的
	* 字符串：当你的正则表达式中没有捕获的组（不分组，或非捕获分组）
	字符串的值，是你的正则表达式所匹配到的单个完整的字符串
```
import re
 
pattern = re.compile(r'\d+')
print re.findall(pattern,'one1two2three3four4')

### 输出 
# ['1', '2', '3', '4']
```
* `finditer(pattern, string, flags=0)`
 返回一个顺序访问每一个匹配结果的迭代器，每个匹配结果返回一个`match`对象。
 用一个`for`循环来接收，然后用`group`来接收。

* `match(pattern, string, flags=0)`
 这个方法将会从string（我们要匹配的字符串）的开头开始，尝试匹配pattern，一直向后匹配，当匹配pattern成功时，返回一个`match`对象，同时匹配终止，不再对string向后匹配。如果遇到无法匹配的字符，立即返回None，如果匹配未结束已经到达string的末尾，也会返回None。两个结果均表示匹配失败。

* `search(pattern, string, flags=0)`
 在字符串中查找，是否能匹配正则表达式。返回`match`对象，如果不能匹配返回None。
 和`match`区别在于match()函数只检测re是不是在string的开始位置匹配，search()会扫描整个string查找匹配，match()只有在0位置匹配成功的话才有返回，如果不是开始位置匹配成功的话，match()就返回None。
 ```
	import re
	 
	# 将正则表达式编译成Pattern对象
	pattern = re.compile(r'world')
	# 使用search()查找匹配的子串，不存在能匹配的子串时将返回None
	# 这个例子中使用match()无法成功匹配
	match = re.search(pattern,'hello world!')
	if match:
	    # 使用Match获得分组信息
	    print match.group()
	### 输出 ###
	# world
 ```
* `purge()`
 清除正则缓存。

* `split(pattern, string, maxsplit=0, flags=0)`
 按照能够匹配的子串将string分割后返回列表。maxsplit用于指定最大分割次数，不指定将全部分割。

* `sub(pattern, repl, string, count=0, flags=0)`
 使用repl替换string中每一个匹配的子串后返回替换后的字符串。
 
 rep1有两种形式：
 当repl是一个字符串时，可以使用\id或\g、\g引用分组，但不能使用编号0。
 当repl是一个方法时，这个方法应当只接受一个参数（Match对象），并返回一个字符串用于替换（返回的字符串中不能再引用分组）。

 count用于指定最多替换次数，不指定时全部替换。
 ```
	import re

	pattern = re.compile(r'(\w+) (\w+)')
	s = 'i say, hello world!'

	print re.sub(pattern,r'\2 \1', s)

	def func(m):
	    return m.group(1).title() + ' ' + m.group(2).title()

	print re.sub(pattern,func, s)

	### output ###
	# say i, world hello!
	# I Say, Hello World!
 ```

* `subn(pattern, repl, string, count=0, flags=0)`
 与re.sub方法作用一样，但返回的是包含新字符串和替换执行次数的两元组。


另外，参数flag是匹配模式，取值可以使用按位或运算符’|’表示同时生效，比如`re.I | re.M`。

可选值有：
> • re.I(全拼：IGNORECASE): 忽略大小写（括号内是完整写法，下同）
 • re.M(全拼：MULTILINE): 多行模式，改变'^'和'$'的行为（参见上图）
 • re.S(全拼：DOTALL): 点任意匹配模式，改变'.'的行为
 • re.L(全拼：LOCALE): 使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
 • re.U(全拼：UNICODE): 使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
 • re.X(全拼：VERBOSE): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。

### match对象

下面我们说一下关于match对象的的属性和方法。

我们在调用re模块的方法时，得到了match对象result，之后我们输出结果用的是result.group()，这个是什么意思呢？

Match对象是一次匹配的结果，包含了很多关于此次匹配的信息，可以使用Match提供的可读属性或方法来获取这些信息。

#### 属性
1.string: 匹配时使用的文本。

2.re: 匹配时使用的Pattern对象。

3.pos: 文本中正则表达式开始搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。

4.endpos: 文本中正则表达式结束搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。

5.lastindex: 最后一个被捕获的分组在文本中的索引。如果没有被捕获的分组，将为None。

6.lastgroup: 最后一个被捕获的分组的别名。如果这个分组没有别名或者没有被捕获的分组，将为None。

#### 方法
1.group([group1, …]):
获得一个或多个分组截获的字符串；指定多个参数时将以元组形式返回。group1可以使用编号也可以使用别名；编号0代表整个匹配的子串；不填写参数时，返回group(0)；没有截获字符串的组返回None；截获了多次的组返回最后一次截获的子串。

2.groups([default]):
以元组形式返回全部分组截获的字符串。相当于调用group(1,2,…last)。default表示没有截获字符串的组以这个值替代，默认为None。

3.groupdict([default]):
返回以有别名的组的别名为键、以该组截获的子串为值的字典，没有别名的组不包含在内。default含义同上。

4.start([group]):
返回指定的组截获的子串在string中的起始索引（子串第一个字符的索引）。group默认值为0。

5.end([group]):
返回指定的组截获的子串在string中的结束索引（子串最后一个字符的索引+1）。group默认值为0。

6.span([group]):
返回(start(group), end(group))。

7.expand(template):
将匹配到的分组代入template中然后返回。template中可以使用\id或\g、\g引用分组，但不能使用编号0。\id与\g是等价的；但\10将被认为是第10个分组，如果你想表达\1之后是字符’0’，只能使用\g0。
---
title: JavaScript学习(2)数据类型转换
date: 2016-08-24 22:14:50
tags: 
- JavaScript
categories: 总结
---

## JavaScript基本语法

### 变量
```
var a = 1;
```
上面的代码先声明变量a，然后在变量a与数值1之间建立引用关系，也称为将数值1“赋值”给变量a。以后，引用变量a就会得到数值1。最前面的var，是变量声明命令。它表示通知解释引擎，要创建一个变量a。

<!-- more -->
JavaScirpt是一种动态类型语言，也就是说，变量的类型没有限制，可以赋予各种类型的值。

可重新赋值，但无需重新声明。

#### null和undefined
如果只是声明变量而没有赋值，则该变量的值是`undefined`。`undefined`是一个JavaScript关键字，表示“无定义，不存在的值”。而不是`null`(理解为无值)。

`null`表示一个“空”的值，它和0以及空字符串''不同，0是一个数值，''表示长度为0的字符串，而`null`表示“空”。

在其他语言中，也有类似JavaScript的`null`的表示，例如Java也用`null`，Swift用`nil`，Python用`None`表示。但是，在JavaScript中，还有一个和`null`类似的`undefined`，它表示“未定义”。

JavaScript的设计者希望用`null`表示一个空的值，而`undefined`表示值未定义。事实证明，这并没有什么卵用，区分两者的意义不大。大多数情况下，我们都应该用`null`。`undefined`仅仅在判断函数参数是否传递的情况下有用。


### 变量提升

JavaScript引擎的工作方式是，先解析代码，获取所有被声明的变量，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

### 标识符

标识符（identifier）是用来识别具体对象的一个名称。

标识符命名规则：

>第一个字符，可以是任意Unicode字母（包括英文字母和其他语言的字母），以及美元符号（$）和下划线（_）。
>第二个字符及后面的字符，除了Unicode字母、美元符号和下划线，还可以用数字0-9

注意不要用保留字，有`arguments、break、case、catch、class、const、continue、debugger、default、delete、do、else、enum、eval、export、extends、false、finally、for、function、if、implements、import、in、instanceof、interface、let、new、null、package、private、protected、public、return、static、super、switch、this、throw、true、try、typeof、var、void、while、with、yield。`

另外，还有三个词虽然不是保留字，但是因为具有特别含义，也不应该用作标识符。分别是`Infinity(无限大，当数值超过了JavaScript的Number所能表示的最大值时，就表示为Infinity)、NaN(表示Not a Number，当无法计算结果时用NaN表示)、undefined(未定义)。`

### 注释符
```
// 这是单行注释

/*
 这是
 多行
 注释
*/
```
此外，由于历史上JavaScript兼容HTML代码的注释，所以`<\!--和-->`也被视为单行注释。

### 区块

JavaScript使用大括号，将多个相关的语句组合在一起，称为“区块”（block）。

与大多数编程语言不一样，JavaScript的区块不构成单独的作用域（scope）。也就是说，区块中的变量与区块外的变量，属于同一个作用域。


### 标签(label)

JavaScript语言允许，语句的前面有标签（label），相当于定位符，用于跳转到程序的任意位置，标签的格式如下。
```
label:
  statement
```

## 数据类型转换

JavaScript是一种动态类型语言，变量没有类型限制，可以随时赋予任意值。虽然变量没有类型，但是数据本身和各种运算符是有类型的。如果运算符发现，数据的类型与预期不符，就会自动转换类型。当然有些时候也会出现需要强制转换的时候。

### 强制转换

强制转换主要指使用`Number`、`String`和`Boolean`三个构造函数，手动将各种类型的值，转换成数字、字符串或者布尔值。

#### Number()

可以将任意类型的值转化成数值。
##### (1)原始类型值的转换规则
```
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('\t\v\r324\n') // 324

// 字符串：如果不可以被解析为数值，返回NaN
Number('abc') // NaN

// 空字符串转为0
Number('') // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0
Number(null) // 0

parseInt('42 abc') // 42

```
`parseInt`逐个解析字符，而`Number`函数整体转换字符串的类型。
另外，`Number`函数会自动过滤一个字符串前导和后缀的空格。

##### (2)对象的转换规则

Number方法的参数是对象时，将返回NaN。

>1.调用对象自身的valueOf方法。如果返回原始类型的值，则直接对该值使用Number函数，不再进行后续步骤。

>2.如果valueOf方法返回的还是对象，则改为调用对象自身的toString方法。如果返回原始类型的值，则对该值使用Number函数，不再进行后续步骤。

>3.如果toString方法返回的是对象，就报错。

#### String()

可以将任意类型的值转化成字符串。

##### (1)原始类型值的转换规则

>数值：转为相应的字符串。
>字符串：转换后还是原来的值。
>布尔值：true转为"true"，false转为"false"。
>undefined：转为"undefined"。
>null：转为"null"。


##### String()对象的转换规则

`String`方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

```
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

`String`方法背后的转换规则，优先`toString`方法，后`valueOf`方法。

#### Boolean()

使用Boolean函数，可以将任意类型的变量转为布尔值。

它的转换规则相对简单：除了以下六个值的转换结果为false，其他的值全部为true。
```
undefined
null
-0
0或+0
NaN
''（空字符串）
```

**注意**:所有对象（包括空对象）的转换结果都是true，甚至连false对应的布尔对象new Boolean(false)也是true。


### 自动转换

下面介绍自动转换，它是以强制转换为基础的。

遇到以下三种情况时，JavaScript会自动转换数据类型，即转换是自动完成的，对用户不可见。

```
// 1. 不同类型的数据互相运算
123 + 'abc' // "123abc"

// 2. 对非布尔值类型的数据求布尔值
if ('abc') {
  console.log('hello')
}  // "hello"

// 3. 对非数值类型的数据使用一元运算符（即“+”和“-”）
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```
自动转换的规则是这样的：预期什么类型的值，就调用该类型的转换函数。比如，某个位置预期为字符串，就调用String函数进行转换。如果该位置即可以是字符串，也可能是数值，那么默认转为数值。

由于自动转换具有不确定性，而且不易出错，建议在预期为布尔值、数值、字符串的地方，全部使用Boolean、Number和String函数进行显式转换。

除了加法运算符有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。
```
false / '5' // 0
'abc' - 1   // NaN
```
----
学习参考：
[廖雪峰的JavaScript教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000)
[JavaScript 标准参考教程 阮一峰](http://javascript.ruanyifeng.com/)
[李炎恢的JavaScript视频教程](http://www.ycku.com/javascript/?utm_source=caibaojian.com)
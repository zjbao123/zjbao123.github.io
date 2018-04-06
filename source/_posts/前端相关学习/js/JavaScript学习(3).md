---
title: JavaScript学习(3)错误处理机制
date: 2016-08-31 22:14:50
tags: 
- JavaScript
categories: 总结
---

## Error对象

JavaScript解析或执行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript原生提供一个Error构造函数，所有抛出的错误都是这个构造函数的实例。
<!-- more -->
```
var err = new Error('出错了');
err.message // "出错了"
```
上面代码中，我们调用Error构造函数，生成一个err实例。

Error构造函数接受一个参数，表示错误提示，可以从实例的message属性读到这个参数。

Error对象的实例必须有message属性，表示出错时的提示信息，其他属性则没有提及。大多数JavaScript引擎，对Error实例还提供name和stack属性，分别表示错误的名称和错误的堆栈，但它们是非标准的，不是每种实现都有。

利用name和message这两个属性，可以对发生什么错误有一个大概的了解。

```
if (error.name){
  console.log(error.name + ": " + error.message);
}
```

stack属性用来查看错误发生时的堆栈。

## JavaScript的原生错误类型
Error对象是最一般的错误类型，在它的基础上，JavaScript还定义了其他6种错误，也就是说，存在Error的6个派生对象。

### （1）SyntaxError

SyntaxError是解析代码时发生的语法错误。

```
// 变量名错误
var 1a;

// 缺少括号
console.log 'hello');
```
###（2）ReferenceError

ReferenceError是引用一个不存在的变量时发生的错误。

ReferenceError: unknownVariable is not defined

另一种触发场景是，将一个值分配给无法分配的对象，比如对函数的运行结果或者this赋值。


###（3）RangeError

RangeError是当一个值超出有效范围时发生的错误。主要有几种情况，一是数组长度为负数，二是Number对象的方法参数超出范围，以及函数堆栈超过最大值。

###（4）TypeError

TypeError是变量或参数不是预期类型时发生的错误。比如，对字符串、布尔值、数值等原始类型的值使用new命令，就会抛出这种错误，因为new命令的参数应该是一个构造函数。

###（5）URIError

URIError是URI相关函数的参数不正确时抛出的错误，主要涉及encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数。

###（6）EvalError

eval函数没有被正确执行时，会抛出EvalError错误。该错误类型已经不再在ES5中出现了，只是为了保证与以前代码兼容，才继续保留。

以上这6种派生错误，连同原始的Error对象，都是构造函数。开发者可以使用它们，人为生成错误对象的实例。

上面的实例可以看到，错误对象的构造函数接受一个参数，代表错误提示信息（message）。

##　自定义错误
除了JavaScript内建的7种错误对象，还可以定义自己的错误对象。
```
function UserError(message) {
   this.message = message || "默认信息";
   this.name = "UserError";
}

UserError.prototype = new Error();
UserError.prototype.constructor = UserError;
```
然后，就可以生成这种自定义的错误了。
```
new UserError("这是自定义的错误！");
```
##　throw语句

throw语句的作用是1.中断程序执行2.抛出一个意外或错误。它接受一个表达式作为参数，可以抛出各种值，包括字符串，数值，布尔值，对象等。

如果只是简单的错误，返回一条出错信息就可以了，但是如果遇到复杂的情况，就需要在出错以后进一步处理。这时最好的做法是使用throw语句自定义抛出一个Error对象。

```
function UserError(message) {
  this.message = message || "默认信息";
  this.name = "UserError";
}

UserError.prototype.toString = function (){
  return this.name + ': "' + this.message + '"';
}

throw new UserError("出错了！");
```
##　try…catch结构

为了对错误进行处理，需要使用try...catch结构。

try代码块一抛出错误（上例用的是throw语句），JavaScript引擎就立即把代码的执行，转到catch代码块。可以看作，错误可以被catch代码块捕获。catch接受一个参数，表示try代码块抛出的值。

catch代码块捕获错误之后，程序不会中断，会按照正常流程继续执行下去。可循环嵌套try...catch

为了捕捉不同类型的错误，catch代码块之中可以加入判断语句。
```
try {
  foo.bar();
} catch (e) {
  if (e instanceof EvalError) {
    console.log(e.name + ": " + e.message);
  } else if (e instanceof RangeError) {
    console.log(e.name + ": " + e.message);
  }
  // ...
}
```
try...catch结构是JavaScript语言受到Java语言影响的一个明显的例子。这种结构多多少少是对结构化编程原则一种破坏，处理不当就会变成类似goto语句的效果，应该谨慎使用。

## finally 代码块

try...catch结构允许在最后添加一个finally代码块，表示不管是否出现错误，都必需在最后运行的语句。

```
function idle(x) {
  try {
    console.log(x);
    return 'result';
  } finally {
    console.log("FINALLY");
  }
}

idle('hello')
// hello
// FINALLY
// "result"
```
即使有return语句在前，finally代码块依然会得到执行，且在其执行完毕后，才会显示return语句的值。

也就是说return先执行后返回，finally后执行先返回。

```
var count = 0;
function countUp() {
  try {
    return count;
  } finally {
    count++;
  }
}

countUp()
// 0
count
// 1
```
上面代码很好的说明了上述情况。

下面是一道经典例题：
```
function f() {
  try {
    console.log(0);
    throw "bug";
  } catch(e) {
    console.log(1);
    return true; // 这句原本会延迟到finally代码块结束再执行
    console.log(2); // 不会运行
  } finally {
    console.log(3);
    return false; // 这句会覆盖掉前面那句return
    console.log(4); // 不会运行
  }

  console.log(5); // 不会运行
}

var result = f();
// 0
// 1
// 3

result
// false
```
从catch转入finally的标志，不仅有return语句，还有throw语句。

```
function f() {
  try {
    throw '出错了！';
  } catch(e) {
    console.log('捕捉到内部错误');
    throw e; // 这句原本会等到finally结束再执行
  } finally {
    return false; // 直接返回
  }
}

try {
  f();
} catch(e) {
  // 此处不会执行
  console.log('caught outer "bogus"');
}

//  捕捉到内部错误
```
上面代码中，进入catch代码块之后，一遇到throw语句，就会去执行finally代码块，其中有return false语句，因此就直接返回了，不再会回去执行catch代码块剩下的部分了。

----
学习参考：
[JavaScript 标准参考教程 阮一峰](http://javascript.ruanyifeng.com/)
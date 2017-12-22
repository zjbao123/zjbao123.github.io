---
title: 初探python(8)错误、调试与测试
date: 2016-10-12 21:26:30
tags: 
- python
- python教程
categories: 总结
---



程序运行过程中会遇到各种错误。

有的是由于程序编写出现问题的，这种错误通常称之为bug，这是必须修复的。

有的是由于用户输入错误造成的，这种错误可以检查用户输入来做相应处理。

还有一类错误是完全无法在程序运行过程中预测的，比如写入文件的时候，磁盘满了，写不进去了，或者从网络抓取数据，网络突然断掉了。这类错误也称为异常，在程序中通常是必须处理的，否则，程序会因为各种问题终止并退出。
<!-- more -->
Python内置了一套异常处理机制，来帮助我们进行错误处理。

此外，我们也需要跟踪程序的执行，查看变量的值是否正确，这个过程称为调试。Python的pdb可以让我们以单步方式执行代码。

最后，编写测试也很重要。有了良好的测试，就可以在程序修改后反复运行，确保程序输出符合我们编写的测试。


## 错误处理

在程序运行的过程中，如果发生了错误，可以事先约定返回一个错误代码，这样，就可以知道是否有错，以及出错的原因。

高级语言通常都内置了一套`try...except...finally...`的错误处理机制，Python也不例外。

### try

当我们认为某些代码可能会出错时，就可以用`try`来运行这段代码，如果执行出错，则后续代码不会继续执行，而是直接跳转至错误处理代码，即`except`语句块，执行完`except`后，如果有`finally`语句块，则执行`finally`语句块，至此，执行完毕。
```
try:
    print "try..."
    r = 10/0
    print 'result:',r
except ZeroDivisionError,e:
    print 'except:',e
finally:
    print 'finally...'
```
如果没有出现错误，`except`不会被执行，但是finally如果有，则一定会被执行（可以没有finally语句）。

当然，错误可以有多个种类，可以添加多个`except`，此外，如果没有错误发生，可以在except语句块后面加一个else，当没有错误发生时，会自动执行else语句。

其实，Python的错误也是class，所有的错误类型都继承自BaseException，所以在使用except时需要注意的是，它不但捕获该类型的错误，还把其子类也“一网打尽”。

[常见错误类型以及继承关系](https://docs.python.org/2/library/exceptions.html#exception-hierarchy)

使用`try...except`捕获错误还有一个巨大的好处，就是可以跨越多层调用，比如函数main()调用foo()，foo()调用bar()，结果bar()出错了，这时，只要main()捕获到了，就可以处理。

这样，也就减少了工作量，不需要在每个地方都去捕获错误，在适当的层次捕获即可。

###　调用堆栈

如果一个错误没有被捕获，它会一直往上抛，最后被Python解释器捕获，打印一个错误信息，然后程序退出。
```
def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    bar('0')

main()

#Traceback (most recent call last):
#  File "err.py", line 11, in <module>
#    main()
#  File "err.py", line 9, in main
#    bar('0')
#  File "err.py", line 6, in bar
#    return foo(s) * 2
#  File "err.py", line 3, in foo
#    return 10 / int(s)
#ZeroDivisionError: integer division or modulo by zero
```

根据错误的跟踪信息，找到对应的错误所在位置。

### 记录错误

如果不捕获错误，自然可以让Python解释器来打印出错误堆栈，但程序也被结束了。既然我们能捕获错误，就可以把错误堆栈打印出来，然后分析错误原因，同时，让程序继续执行下去。

Python内置的`logging`模块可以非常容易地记录错误信息:
```
import logging

def foo(s):
    return 10 / int(s)

def bar(s):
    return foo(s) * 2

def main():
    try:
        bar('0')
    except StandardError, e:
        logging.exception(e)

main()
print 'END'
```
通过配置，`logging`还可以把错误记录到日志文件里，方便事后排查。

### 抛出错误

因为错误是class，捕获一个错误就是捕获到该class的一个实例。因此，错误并不是凭空产生的，而是有意创建并抛出的。Python的内置函数会抛出很多类型的错误，我们自己编写的函数也可以抛出错误。我们可以通过`raise`来抛出错误的实例。

有些时候，尽管捕获到了错误，可捕获错误目的只是记录一下，便于后续追踪。但是，由于当前函数不知道应该怎么处理该错误，所以，最恰当的方式是继续往上抛，让顶层调用者去处理。

`raise`如果不带参数，就将错误原样直接抛出；也可以在后面加一个`Error`，来将错误类型进行转化。当然，前提是要符合逻辑。

## 调试
程序很难一次性写对，因此，我们需要进行调试。
调试有四种方式：
1.print
2.assert
3.logging
4.单步调试

第一种简单粗暴，直接打印出来看看到底是什么。

第二种可以替代`print`， 例如：`assert n != 0, 'n is zero!'`
`assert`的意思是，表达式`n != 0`应该是`True`，否则，后面的代码就会出错。

如果断言失败，`assert`语句本身就会抛出`AssertionError：n is zero!`

不过，启动Python解释器时可以用-O参数来关闭assert。关闭后，你可以把所有的assert语句当成pass来看。

```
 python -O err.py
```

第三种就是logging，允许输出一行文本。
```
__author__ = 'zjbao123'
import logging
logging.basicConfig(level=logging.INFO)

s = '3'
n = int(s)
logging.info('n = %d' % n)
print 10 / n
```
这就是`logging`的好处，它允许你指定记录信息的级别，有debug，info，warning，error等几个级别，指定高级别的时候，低级别就不起作用当我们指定`level=INFO`时，`logging.debug`就不起作用了。这样一来，你可以放心地输出不同级别的信息，也不用删除，最后统一控制输出哪个级别的信息。

另外，就是单步调试了，利用IDE来设置断点进行单步调试。

## 单元测试

单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作。

当你修改模块内容的时候，单元测试仍然可用。

单元测试可以有效地测试某个程序模块的行为，是未来重构代码的信心保证。

单元测试的测试用例要覆盖常用的输入组合、边界条件和异常。

这种以测试为驱动的开发模式最大的好处就是确保一个程序模块的行为符合我们设计的测试用例。在将来修改的时候，可以极大程度地保证该模块行为仍然是正确的。
```
class test_abs(unittest.TestCase):
    def test_value(self):
        d = abs(-1)
        self.assertEqual(d,1)

    def test_abserror(self):
        with self.assertRaises(TypeError):
           abs('sd')
    def setUp(self):
        print 'setUp...'

    def tearDown(self):
        print 'tearDown...'

if __name__ == '__main__':
    unittest.main()
```
###　单元测试的写法

编写单元测试时，我们需要编写一个测试类，从unittest.TestCase继承。

以`test`开头的方法就是测试方法，不以`test`开头的方法不被认为是测试方法，测试的时候不会被执行。

对每一类测试都需要编写一个test_xxx()方法。由于unittest.TestCase提供了很多内置的条件判断，我们只需要调用这些方法就可以断言输出是否是我们所期望的。最常用的断言就是assertEquals().

另一种重要的断言就是期待抛出指定类型的Error,如上例子所述。

### 运行单元测试

最简单的运行方式是在mydict_test.py的最后加上两行代码：
```
if __name__ == '__main__':
    unittest.main()
```

另一种更常见的方法是在命令行通过参数-m unittest直接运行单元测试

```
python -m unittest mydict_test
```

这是推荐的做法，因为这样可以一次批量运行很多单元测试，并且，有很多工具可以自动来运行这些单元测试。


###　setUp与tearDown

这两个方法会分别在每调用一个测试方法的前后分别被执行。`setUp()`和`tearDown()`方法有什么用呢？设想你的测试需要启动一个数据库，这时，就可以在`setUp()`方法中连接数据库，在`tearDown()`方法中关闭数据库，这样，不必在每个测试方法中重复相同的代码。


## 文档测试

在官方文档中，有很多文档都有实例代码。这些代码与其他说明可以写在注释中，然后，由一些工具来自动生成文档。既然这些代码本身就可以粘贴出来直接运行，那么，可不可以自动执行写在注释中的这些代码呢？

答案是肯定的。

当我们编写注释时，如果写上这样的注释：
```
def abs(n):
    '''
    Function to get absolute value of number.

    Example:

    >>> abs(-1)
    1
    >>> abs(0)
    0
    >>> abs('sd')
    Traceback (most recent call last):
        ...
    TypeError: bad operand type
    '''
    if isinstance(n,int):
        return n if n >= 0 else (-n)
    else:
        raise TypeError('bad operand type')
if __name__=='__main__':
    import doctest
    doctest.testmod()
```

运行时，什么输出也没有。这说明我们编写的doctest运行都是正确的。如果程序有问题，再运行就会报错。

doctest非常有用，不但可以用来测试，还可以直接作为示例代码。通过某些文档生成工具，就可以自动把包含doctest的注释提取出来。用户看文档的时候，同时也看到了doctest。


>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
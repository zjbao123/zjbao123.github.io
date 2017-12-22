---
title: 使用Distutils分发自己编写的Python包
date: 2017-04-19 21:22:50
tags: 
- 协议
categories: 总结
---

>翻译自[官方文档](https://docs.python.org/2.7/distutils/index.html)

本文档从模块开发人员的角度描述了Python Distribution Utilities（“Distutils”），介绍了如何使用Distutils将Python模块和扩展程序提供给更加广泛的使用者，并使得构建/释放/安装机制的开销很少。

<!--more-->

## 概念&术语

对于模块开发人员和安装第三方模块的用户/管理员来说，使用Distutils是非常简单的。作为一名开发人员，您的责任（当然除了编写可靠的，有详细记录和经过良好测试的代码）是：

* 写一个安装脚本（按照惯例为setup.py）
* （可选）写入设置配置文件
* 建立一个源码分发包
* （可选）创建一个或多个内置（二进制）分发包

这些都会在本文档中一一提到。

并不是所有的模块开发人员都可以访问多种平台，因此总想着他们可以创建大量的内置分发包是不可行的。希望出现一类称为“packagers”的中介机构，以满足这一需求。packagers将采用模块开发人员发布的源代码，将其构建在一个或多个平台上，并发布生成的内置分发包。因此，最流行的平台上的用户将能够以最自然的方式为他们的平台安装大多数流行的Python模块分发包，而无需运行单个安装脚本或编译一行代码。

## 一个简单实例

安装脚本通常非常简单，尽管由Python编写，但是对于可以使用的脚本没有任何限制，可还是要在安装脚本中应该谨慎处理那些重要的操作。 与`Autoconf`风格的配置脚本不同的是，在构建和安装模块分发包的过程中，安装脚本可能会多次运行。

如果你想做的只是分发一个包含在`foo.py`文件中的`foo`模块，那么你的安装脚本就可以这么简单：
```
from distutils.core import setup
setup(name='foo',
      version='1.0',
      py_modules=['foo'],
      )

```
一些需要注意的地方：

* 您提供给`Distutils`的大多数信息都作为关键字参数提供给`setup（）`函数

* 这些关键字参数分为两类：包元数据（名称，版本号）以及有关软件包内容的信息（在这种情况下是纯`Python`模块的列表）

* 模块由模块名称指定，而不是文件名（对于包和扩展名也是如此）

* 建议您提供更多元数据，特别是您的名称，电子邮件地址和项目的URL（请参阅编写安装脚本一节）


如果要将源码进行分发，你需要在`setup.py`写入上述代码，并从命令行处运行：

```
python setup.py sdist
```
对于Windows用户来说，你需要执行：

```
setup.py sdist
```

`sdist`将创建一个存档文件（例如，Unix上的tarball文件，Windows上的ZIP文件），其中包含您的安装脚本`setup.py`和您的模块`foo.py`。 归档文件将被命名为`foo-1.0.tar.gz（或.zip）`，并将打包到目录`foo-1.0`中。

如果最终用户希望安装您的foo模块，那么她所要做的就是下载`foo-1.0.tar.gz（或.zip）`，解压缩，并从`foo-1.0`目录运行：

```
python setup.py install
```

这将最终将`foo.py`复制到Python安装中的第三方模块的相应目录。

这个简单的例子演示了`Distutils`的一些基本概念。 首先，开发人员和安装人员都有相同的基本用户界面，即安装脚本。 区别在于`Distutils`使用的命令：`sdist`命令几乎专门用于模块开发人员，而安装程序更常用于安装程序（尽管大多数开发人员都希望偶尔安装自己的代码）。

如果你想为用户做一些简单的事情，你可以为他们创建一个或多个内置（二进制）的分发包。 例如，如果您正在Windows机器上运行，并希望为其他Windows用户提供方便，则可以使用`bdist_wininst`命令创建一个可执行安装程序（这个平台最合适的构建版本的类型）。 例如：
```
python setup.py bdist_wininst
```

这将在当前目录中创建可执行安装程序`foo-1.0.win32.exe`。

其他有用的内置分发格式是RPM。 例如，以下命令将创建一个名为`foo-1.0.noarch.rpm`的RPM文件：

```
python setup.py bdist_rpm
```

>注：bdist_rpm命令使用rpm可执行文件，因此必须在基于RPM的系统上运行，如Red Hat Linux，SuSE Linux或Mandrake Linux。

```
python setup.py bdist --help-formats
```

你可以通过这条指令来查看运行了什么样的分发格式。

## 常用的Python术语

**模块(module)**:Python中代码可重用性的基本单位：由一些其他代码导入的代码块。 这里涉及到三种类型的模块：纯Python模块（Python modules），扩展模块（extension modules）和包（packages）。

**纯Python模块（Python modules）**：一个用Python编写并包含在一个.py文件中的模块（也可能是关联的.pyc和/或.pyo文件）。 有时被称为“纯模块”。

**扩展模块（extension modules）**：一个用Python实现的低级语言编写的模块：C / C ++ for Python，Java for Jython。 通常包含在单个可动态加载的预编译文件中。 Unix上的Python扩展的共享对象（.so）文件，Windows上的Python扩展的DLL（给定.pyd扩展名）或Jython扩展的Java类文件。 （请注意，目前，Distutils仅处理Python的C / C ++扩展。）

**包（packages）**:包含其他模块的模块; 通常包含在文件系统中的目录中，并通过存在文件`__init__.py`与其他目录区分开来。

**根包（root packages）**：
包的层次结构的根。 （这不是一个包，因为它没有一个`__init__.py`文件，但我们必须称之为一些）。绝大多数的标准库是根包，许多小的，独立的第三个 不属于较大模块集合的一个模块。 与常规包不同，根包中的模块可以在许多目录中找到：实际上，`sys.path`中列出的每个目录都向根包提供模块。

## Distutils特定术语

以下术语更具体地适用于使用`Distutils`分发`Python`模块的方面：

**模块分发包**:作为一个可下载的资源一起分发的Python模块的集合，并且要大量安装。一些众所周知的模块分布的例子有数字Python，PyXML，PIL（Python影像库）或mxBase。 （这将被称为包，除了该术语已经在Python上下文中：单个模块分发可能包含零个，一个或多个Python包。）

**纯模块分发包**：一个仅包含纯Python模块和包的模块分发。有时被称为“纯分配”。

**非纯模块分发包**：一个包含至少一个扩展模块的模块分发。有时被称为“非纯分配”。

**根分发包**：源树（或源分发）的顶层目录; setup.py存在的目录。一般来说，setup.py将从此目录运行。
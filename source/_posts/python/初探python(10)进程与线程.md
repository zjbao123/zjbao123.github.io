---
title: 初探python(10)进程与线程
date: 2016-10-20 16:55:30
tags: 
- python
- python教程
categories: 总结
---


的确，线程和进程的知识有点繁，耐下性子先过一遍吧。
## 什么是多进程和多线程

对于操作系统来说，一个任务就是一个进程（Process）。
多进程的操作系统轮流让各个进程交替执行，由于CPU的执行速度实在是太快了，我们感觉就像所有任务都在同时执行一样。
<!-- more -->
真正的并行执行多进程只能在多核CPU上实现，但是，由于进程数量远远多于CPU的核心数量，所以，操作系统也会自动把很多进程轮流调度到每个核心上执行。

在一个进程内部有些还不是干一件事，像Word可以有多件事同时干，就需要运行多个子任务，我们把子任务称之为“线程”（Thread）。

由于每个进程至少要干一件事，所以至少有一个线程。操作系统在多个线程之间快速切换，让每个线程都短暂地交替运行，看起来就像同时执行一样。当然，真正地同时执行多线程需要多核CPU才可能实现。

那python要同时执行执行多个任务的话，多任务的实现有3种方式：

多进程模式；
多线程模式；
多进程+多线程模式。

多进程和多线程的程序涉及到同步、数据共享的问题，编写起来更复杂。

## 多进程

### fork
Unix/Linux操作系统提供了一个fork()系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。
```
import os

print 'Process (%s) start...' % os.getpid()
pid = os.fork()
if pid==0:
    print 'I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid())
else:
    print 'I (%s) just created a child process (%s).' % (os.getpid(), pid)
```
有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

### multiprocessing

但我用的是Windows系统，由于Python是跨平台的，自然也应该提供一个跨平台的多进程支持。`multiprocessing`模块就是跨平台版本的多进程模块。
```
from multiprocessing import Process
import os

def run_proc(name):
    print 'Run child process %s (%s)...' % (name, os.getpid())
if __name__ == '__main__':
    print 'Run parent process %s...' % os.getpid()
    p = Process(target=run_proc, args=('test',))
    print 'child process will start ...'
    p.start()
    p.join()
    print 'Process end.'
```
创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。

join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

###　Pool
如果要启动大量的子进程，可以用进程池的方式批量创建子进程：
```
from multiprocessing import Pool
import os, time, random

def long_time_task(name):
    print 'Run task %s (%s)...' % (name, os.getpid())
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print 'Task %s runs %0.2f seconds.' % (name, (end - start))

if __name__=='__main__':
    print 'Parent process %s.' % os.getpid()
    p = Pool()
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print 'Waiting for all subprocesses done...'
    p.close()
    p.join()
    print 'All subprocesses done.'

```

由于Pool的默认大小是CPU的核数。task一次性执行4个，其他的要等之前的执行完才能执行，如果改成:
```
p = Pool(5)
```

### 进程间通信

Python的`multiprocessing`模块包装了底层的机制，提供了`Queue`、`Pipes`等多种方式来交换数据。

我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

```
from multiprocessing import Process,Queue
import os,time,random


# 写数据进程执行的代码:
def write(q):
    for value in ['A','B','C']:
        print 'Put %s to queue...' %value
        q.put(value)
        time.sleep(random.random())


# 读数据进程执行的代码:
def read(q):
    while True:
        value = q.get(True)
        print 'Get %s from queue.' % value

if __name__ =='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pw，读出:
    pr.start()
    pw.join()
    # pr进程无限循环，强制终止
    pr.terminate()

```

##　多线程
由于线程是操作系统直接支持的执行单元，因此，python高级语言通常都内置多线程的支持，并且Python的线程是真正的Posix Thread（可移植操作系统接口 线程），而不是模拟出来的线程。

Python的标准库提供了两个模块：thread和threading，thread是低级模块，threading是高级模块，对thread进行了封装。绝大多数情况下，我们只需要使用threading这个高级模块。

启动一个线程就是把一个函数传入并创建Thread实例，然后调用start()开始执行：
```
import time, threading

def loop():
    print 'thread %s is running ... ' % threading.current_thread().name
    n = 0
    while n <5:
        n += 1
        print 'thread %s => %s' % (threading.current_thread().name,n)
        time.sleep(1)
    print 'thread %s ended' % threading.current_thread().name

print 'thread %s is running ...' % threading.current_thread().name
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print 'thread %s ended' % threading.current_thread().name
```
由于任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程，Python的threading模块有个`current_thread()`函数，它永远返回当前线程的实例。主线程实例的名字叫`MainThread`，子线程的名字在创建时指定，我们用`LoopThread`命名子线程。名字仅仅在打印时用来显示，完全没有其他意义，如果不起名字Python就自动给线程命名为`Thread-1`，`Thread-2`……



### Lock
多线程和多进程最大的不同在于:
* 多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响;
* 多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改。

因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

所以，我们必须确保一个线程在修改变量的时候，别的线程一定不能改。
```
import time, threading

balance = 0
lock = threading.Lock()


def change_it(n):
    global balance
    balance = balance - n
    balance = balance + n

def run_thread(n):
    for i in range(1000):
        lock.acquire()
        try:
            change_it(i)
        finally:
            lock.release()

thread1 = threading.Thread(target=run_thread, args=(5,))
thread2 = threading.Thread(target=run_thread, args=(8,))

thread1.start()
thread2.start()
thread1.join()
thread2.join()
print balance
```

### 多核CPU

如果你不幸拥有一个多核CPU，你肯定在想，多核应该可以同时执行多个线程。

如果写一个死循环的话，会出现什么情况呢？

打开Mac OS X的Activity Monitor，或者Windows的Task Manager，都可以监控某个进程的CPU使用率。

我们可以监控到一个死循环线程会100%占用一个CPU。

启动与CPU核心数量相同的N个线程，在4核CPU上可以监控到CPU占用率仅有160%，但是用C、C++或Java来改写相同的死循环，直接可以把全部核心跑满，4核就跑到400%，8核就跑到800%，为什么Python不行呢？

因为Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

GIL是Python解释器设计的历史遗留问题，通常我们用的解释器是官方实现的CPython，要真正利用多核，除非重写一个不带GIL的解释器。

所以，在Python中，可以使用多线程，但不要指望能有效利用多核。如果一定要通过多线程利用多核，那只能通过C扩展来实现，不过这样就失去了Python简单易用的特点。

不过，也不用过于担心，Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响。

## ThreadLocal
正如上面所说，在多线程环境下，对于全局变量的修改必须加锁，因此，一个线程使用局部变量只有线程自己看得到，不会影响其他线程。

但局部变量也有问题。就是在传递的时候非常麻烦。尽管是局部变量，但是每个函数都要用它，因此必须传进去。用全局变量又会影响其他进程，不能共享。
```
global_dict = {}

def std_thread(name):
    std = Student(name)
    # 把std放到全局变量global_dict中：
    global_dict[threading.current_thread()] = std
    do_task_1()
    do_task_2()

def do_task_1():
    # 不传入std，而是根据当前线程查找：
    std = global_dict[threading.current_thread()]
    ...

def do_task_2():
    # 任何函数都可以查找出当前线程的std变量：
    std = global_dict[threading.current_thread()]
```
如果用一个全局变量`dict`来存放所有的`Student`对象，然后以`thread`自身作为`key`获得线程对应的`student`,这样就消除了`std`对象在每层函数中传递问题，但是还是不够简单。

`ThreadLocal`就是来自动帮你做这些事情的，不用查找`dict`。
```
import threading

# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    print 'Hello, %s (in %s)' % (local_school.student, threading.current_thread().name)

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```
全局变量`local_school`是一个`ThreadLocal`对象，每个`Thread`对它都可以读写`student`属性，但互不影响。你可以把`local_school`看成全局变量，但每个属性如`local_school.student`都是线程的局部变量，可以任意读写而不互相干扰，也不用管理锁的问题，`ThreadLocal`内部会进行处理。

可以理解为全局变量`local_school`是一个`dict`，不但可以用`local_school.student`，还可以绑定其他变量，如`local_school.teacher`等等。

`ThreadLocal`最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

### 进程 vs. 线程

现在我们来讨论一下这两种方式的优缺点。

首先，要实现多任务，通常我们会设计`Master-Worker`模式，Master负责分配任务，Worker负责执行任务，因此，多任务环境下，通常是一个Master，多个Worker。

自然，主的就是Master，其他进程/线程就是Worker

多进程最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程（当然主进程挂了所有子进程都挂了，但是Master进程只负责分配任务，挂掉的概率低）。著名的Apache最早就是采用多进程模式。

多进程的缺点就是创建进程的代价大，在Unix/Linu系统下，用`fork`调用还行，在Windows下创建进程就开销巨大。另外，操作系统能同时运行的进程数也是有限的，在内存和CPU的限制下，如果有几千个进程同时运行，操作系统连调度都是问题。

多线程模式通常比多进程要快一些，但是也快不到哪里去，而且，多线程模式致命的缺点就是任何一个线程挂掉都直接可能造成整个进程崩溃，因为所有线程共享进程的内存。在Windows上，如果一个线程执行的代码出现了问题，你经常可以看到这样的提示：“该程序执行了非法操作，即将关闭”，其实往往是某个线程出了问题，但是操作体统会强制结束整个进程。

在Windows下，多线程的效率要比多进程高，所以微软的IIS服务器默认采用多线程模式，为了缓解这个问题，IIS和Apache现在又有多线程+多进程的混合模式，真是把问题越搞越复杂。

### 线程切换

无论是多进程还是多线程，只要数量一多，效率肯定上不去，为什么呢？

任务之间切换是有代价的。比如你从语文切换到数学，要收拾桌子上的语文书（这叫保存现场），然后打开数学课本（这叫准备新环境），才能开始做数学作业。操作系统在切换进程/线程也是需要时间的，它需要先保存当前执行的现场环境(CPU寄存器状态、内存页等)然后，把新任务的执行环境准备好(恢复上次的寄存器状态，切换内存页等)，才能开始执行。这个切换虽然快，但也需要时间，操作系统在任务多的时候可能主要忙着切换任务，根本没有多少时间去执行任务了，这种情况最常见的就是硬盘狂响，点窗口没反应，系统处于假死状态。

所以，多任务一旦多到一个限度，就会消耗掉系统所有的资源，结果笑来急剧下降，所有任务都做不好。

### 计算密集型 vs IO密集型

是否采用多任务的第二个考虑是任务的类型。我们可以吧任务分为计算密集型和IO密集型。

计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要高效利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

计算密集型任务主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务，多余计算密集型任务最好用C语言编写。

第二种任务类型就是IO密集型，涉及到网络、磁盘IO任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成(因为IO的速度远远低于CPU和内存的速度)。对于IO密集型任务，任务越多，CPU效率越高，但也有个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

IO密集型任务执行期间，99%的时间都花在了IO上，花在CPU的时间爱你很少，对于IO密集型任务，最适合的语言就是开发效率最高(代码量最少)的语言，脚本语言是首选，C最差。

### 异步IO
考虑到CPU和IO之间巨大的速度差异，一个任务在执行的过程汇总大部分时间都在等待IO操作，单进程单线程模型会导致别的任务无法并行执行，因此，我们才需要多进程模型或者多线程模型来支持多任务并发执行。

现代操作系统多IO操作已经做了巨大的改进，最大的特点就是支持异步IO，如果充分利用操作系统提供的异步IO支持，就可以用单进程单线程模型来执行多任务，这种全新的模型称为事件驱动模型，Nginx就是支持异步IO的Web服务器，它在单核CPU上采用单进程模型就可以高效的支持多任务。在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。由于系统总的进程数量十分有限，因此操作系统调度非常有效。用异步IO编程模型来实现多任务是一个主要的趋势。

对应到python语言，单进程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。

协程我们后续会提到。

### 分布式进程

在process和thread中，应当优先选process，因为process更稳定，而且，process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。

Python的`multiprocessing`模块不但支持多进程，其中`managers`子模块还支持把多进程分布到多台机器上。一个服务进程可以作为跳度者，将任务分布到其他多个进程中，依靠网络通信。由于`managers`模块封装很好，不必了解网络通信的细节，就可以很容易地编写分布式多进程程序。

举个例子：如果我们已经有一个通过`Queue`通信的多进程程序在同一台机器上运行，现在，由于处理任务的进程任务繁重，希望吧发送任务的进程和处理任务的进程分布到两台机器上。怎么用分布式进程实现？

原有的`Queue`可以继续使用，但是，通过`managers`模块把`Queue`通过网络暴Queue露出去，就可以让其他机器的进程访问`Queue`了。

我们先看服务进程，服务进程负责启动`Queue`，把`Queue`注册到网络上，然后往`Queue`里面写任务：
```
'分布式进程 -- 服务器端'

import random, multiprocessing
from multiprocessing.managers import BaseManager
from multiprocessing import freeze_support


# 从BaseManager继承的QueueManager:
class QueueManager(BaseManager):
    pass


# 发送任务的队列:
task_queue = multiprocessing.Queue()
# 接收结果的队列:
result_queue = multiprocessing.Queue()

# 为解决__main__.<lambda> not found问题
def get_task_queue():
    return task_queue

# 为解决__main__.<lambda> not found问题
def get_result_queue():
    return result_queue


# 把两个Queue都注册到网络上, callable参数关联了Queue对象:
QueueManager.register('get_task_queue', callable=get_task_queue)
QueueManager.register('get_result_queue', callable=get_result_queue)
# 绑定端口5000, 设置验证码'abc':
manager = QueueManager(address=('127.0.0.1', 5000), authkey='abc')


def communicate():
    # 获得通过网络访问的Queue对象:
    task = manager.get_task_queue()
    result = manager.get_result_queue()

    # 放几个任务进去:
    for i in range(10):
        n = random.randint(0, 10000)
        print('Put task %d...' % n)
        task.put(n)

    # 从result队列读取结果:
    print('Try get results...')
    for i in range(10):
        r = result.get(timeout=10)
        print('Result: %s' % r)

    # 关闭:
    manager.shutdown()


if __name__ == '__main__':
    freeze_support()
    # 启动Queue:
    manager.start()
    communicate()

```
请注意，当我们在一台机器上写多进程程序时，创建的Queue可以直接拿来用，但是，在分布式多进程环境下，添加任务到`Queue`不可以直接对原始的`task_queue`进行操作，那样就绕过了`QueueManager`的封装，必须通过`manager.get_task_queue()`获得的`Queue`接口添加。
```
import time, sys, Queue
from multiprocessing.managers import BaseManager

# 创建类似的QueueManager:
class QueueManager(BaseManager):
    pass

# 由于这个QueueManager只从网络上获取Queue，所以注册时只提供名字:
QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')

# 连接到服务器，也就是运行taskmanager.py的机器:
server_addr = '127.0.0.1'
print('Connect to server %s...' % server_addr)
# 端口和验证码注意保持与taskmanager.py设置的完全一致:
m = QueueManager(address=(server_addr, 5000), authkey='abc')
# 从网络连接:
m.connect()
# 获取Queue的对象:
task = m.get_task_queue()
result = m.get_result_queue()
# 从task队列取任务,并把结果写入result队列:
for i in range(10):
    try:
        n = task.get(timeout=1)
        print('run task %d * %d...' % (n, n))
        r = '%d * %d = %d' % (n, n, n*n)
        time.sleep(1)
        result.put(r)
    except Queue.Empty:
        print('task queue is empty.')
# 处理结束:
print('worker exit.')
```
任务进程要通过网络连接到服务进程，所以要指定服务进程的IP。

先执行服务器端，再执行worker端，可以实现master端和worker端的分布式计算了。

另外，发现`worker.py`根本没有创建`Queue`，所以，`Queue`对象存储在`manager.py`进程中。

而`queue`之所以可以通过网络访问，是通过`queueManager`实现的。由于`queueManager`管理的不知一个`Queue`，所以，要给每个Queue的网络调用接口起个名字，比如`get_task_queue`。

而`authkey`为了保证两台机器之间正常通信，不被其他机器干扰，做验证码用。

注意`Queue`的作用是用来传递任务和接收结果，每个任务的描述数据量要尽量小。比如发送一个处理日志文件的任务，就不要发送几百兆的日志文件本身，而是发送日志文件存放的完整路径，由Worker进程再去共享的磁盘上读取文件。

>此python学习路径来源于[廖雪峰的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)的一个学习内容的总结。以便于自己后的学习和整理。
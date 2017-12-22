---
title: OPC开发小结
date: 2016-05-21 20:51:02
tags:
- c#
- mysql
- 串口
categories: 总结
---

总算完成了这个小项目的开发，是时候总结一下用过的东西了。这个小项目完成了什么功能呢？通过一个c#的窗口程序来访问`OPC`底层的数据然后读取并上传到mysql数据库中进行保存。考虑到有可能运行该程序的电脑无法联网，于是增加了通过串口来传输数据。

<!-- more -->

## 什么是OPC

`OPC`(OLE for Process Control, 用于过程控制的OLE)是一个工业标准，管理这个标准国际组织是`OPC`基金会，`OPC`基金会现有会员已超过220家。遍布全球，包括世界上所有主要的自动化控制系统、仪器仪表及过程控制系统的公司。基于微软的OLE(现在的Active X)、COM （部件对象模型）和DCOM （分布式部件对象模型）技术。`OPC`包括一整套接口、属性和方法的标准集，用于过程控制和制造业自动化系统。
`OPC`是为了连接数据源(`OPC`服务器)和数据的使用者(`OPC`应用程序)之间的软件接口标准。简单的说，就是连接硬件的驱动器和与其连接的应用程序之间的接口的一个统一的标准。通过`OPC`就可以访问到底层的数据啦！
`OPC`客户端源码本身就已经是很早以前的技术了，网上能找到的都在2012年左右，原型采用[大尾巴狼啊](http://www.cnblogs.com/badnewfish/archive/2009/04/11/1374966.html)提供的源码(原博客被删了，只能找到转载的了)，对于初学者真的作用很大。

##  OPC数据定时读取

由于opc数据实时更新，需要定时读取数据。因此在原有的程序上增加了定时器timer。采用的是`System.Timers.Timer`。


```
  private void initializetimer()//初始化定时器
        {
            Timers_Timer.Interval = 5000;//每5秒读取一次
            Timers_Timer.Enabled = true;
            Timers_Timer.Elapsed += new System.Timers.ElapsedEventHandler(Timers_Timer_Elapsed);
            Timers_Timer.AutoReset = true;//自动重启
        }
  void Timers_Timer_Elapsed(object sender, System.Timers.ElapsedEventArgs e)
        {
        	//添加定时器到时后所需运行的代码
        }
```
## OPC数据异步读

之后采用的`OPC`异步读取参考了[OPCDAAuto.dll的C#使用方法浅析](http://www.cnblogs.com/zjjking/archive/2009/01/23/1380218.html)，需注意的是，Array ServerHandle 数组从下标1开始而非0！之后调用 `AsyncRead` 异步读方法。

![异步读写](http://7xsp7y.com2.z0.glb.qiniucdn.com/16-5-26/55772370.jpg)

异步通讯时，OPC客户程序对服务器进行请求时，OPC客户程序请求后立刻返回，不用等待OPC服务器响应，可以进行其他操作。OPC服务完成响应后，再通知客户端进行读操作，此时调用`GroupAsyncReadComplete`，来进行后续的写入mysql程序。

```cs
    KepGroup.AsyncRead(listBox1.Items.Count, ref serverHandles, out Errors, 1, out cancelID);

    void GroupAsyncReadComplete(int TransactionID, int NumItems, ref System.Array ClientHandles, ref System.Array ItemValues, ref System.Array Qualities, ref System.Array TimeStamps, ref System.Array Errors)
    {
    	//添加后续写入mysql程序
    }
```

##　C#连接mysql数据库

由于用到的是阿里云服务器下搭建的数据库，采用`navicat`进行远程连接并数据库管理。由于C#
连接mysql数据库需添加[mysql.data.dll](http://hovertree.com/h/bjaf/0sft36s9.htm)引用。
下载Mysql.Data.dll，然后在项目中添加该组件的引用，在代码页里输入using Mysql.Data.MysqlClient，我们就可以顺利的使用该类库的函数建立连接了。
实现实例代码：

 ```cs
 string M_str_sqlcon = "server=localhost;user id=root;password=root;database=abc"; //根据自己的设置
 MySqlConnection myCon = new MySqlConnection(M_str_sqlcon);
 mysqlcon.Open();
 MySqlCommand mysqlcom = new MySqlCommand(M_str_sqlstr, mysqlcon);//M_str_sqlstr 为sql语句，需另外定义
 mysqlcom.ExecuteNonQuery();
 mysqlcom.Dispose();
 mysqlcon.Close();
 mysqlcon.Dispose();
```
之后便可以用`navicat`远程连接到数据库进行查看。

## opc数据用串口发送

### 什么是串口
串口是计算机上一种非常通用设备通信的协议(不要与通用串行总线Universal Serial Bus或者USB混淆).大多数计算机包含两个基于RS232的串口.串口同时也是仪器仪表设备通用的通信协议;很多GPIB兼容的设备也带有RS-232口.同时,串口通信协议也可以用于获取远程采集设备的数据.串口通信的概念非常简单，串口按位(bit)发送和接收字节。尽管比按字节（byte）的并行通信慢，但是串口可以在使用一根线发送数据的同时用另一根线接收数据。它很简单并且能够实现远距离通信。比如IEEE488定义并行通行状态时，规定设备线总常不得超过20米，并且任意两个设备间的长度不得超过2米；而对于串口而言，长度可达1200米。 典型地，串口用于ASCII码字符的传输。通信使用3根线完成：（1）地线，（2）发送，（3）接收。由于串口通信是异步，端口能够在一根线上发送数据同时在另一根线上接收数据。其他线用于握手，但是不是必须的。串口通信最重要的参数是波特率、数据位、停止位和奇偶校验。对于两个进行通行的端口，这些参数必须匹配。
MS在.NET FrameWork2.0中对串口通讯进行了封装，我们可以在.net2.0及以上版本开发时直接使用SerialPort类对串口进行读写操作。

接下来讲讲几个比较重要的属性：

**串口名称 PortName**

通信的端口名称，例如COM1,COM2 之类

**波特率 BaudRate**

用来衡量通信速度的参数。它表示每秒钟传送的bit的个数。例如9600波特表示每秒钟发送9600个bit。当我们提到时钟周期时，我们就是指波特率例如如果协议需要4800波特率，那么时钟是4800Hz。这意味着串口通信在数据线上的采样率为4800Hz。通常电话线的波特率为14400，28800和36600。波特率可以远远大于这些值，但是波特率和距离成反比。高波特率常常用于放置的很近的仪器间的通信，典型的例子就是GPIB设备的通信。

**数据位 DataBits**

这是衡量通信中实际数据位的参数。当计算机发送一个信息包，实际的数据不会是8位的，标准的值是5、7和8位。如何设置取决于你想传送的信息。比如，标准的ASCII码是0～127（7位）。扩展的ASCII码是0～255（8位）。如果数据使用简单的文本（标准 ASCII码），那么每个数据包使用7位数据。每个包是指一个字节，包括开始/停止位，数据位和奇偶校验位。由于实际数据位取决于通信协议的选取，术语“包”指任何通信的情况。

**停止位 StopBits** 

用于表示单个包的最后一位。典型的值为1，1.5和2位。由于数据是在传输线上定时的，并且每一个设备有其自己的时钟，很可能在通信中两台设备间出现了小小的不同步。因此停止位不仅仅是表示传输的结束，并且提供计算机校正时钟同步的机会。适用于停止位的位数越多，不同时钟同步的容忍程度越大，但是数据传输率同时也越慢。

**奇偶校验 Parity**

奇偶校验位：在串口通信中一种简单的检错方式。有四种检错方式：偶、奇、高和低。当然没有校验位也是可以的。对于偶和奇校验的情况，串口会设置校验位（数据位后面的一位），用一个值确保传输的数据有偶个或者奇个逻辑高位。例如，如果数据是011，那么对于偶校验，校验位为0，保证逻辑高的位数是偶数个。如果是奇校验，校验位位1，这样就有3个逻辑高位。高位和低位不真正的检查数据，简单置位逻辑高或者逻辑低校验。这样使得接收设备能够知道一个位的状态，有机会判断是否有噪声干扰了通信或者是否传输和接收数据是否不同步
MSDN 讲解http://msdn.microsoft.com/zh-cn/library/system.io.ports.parity.aspx
关于奇偶校验的讲解http://baike.baidu.com/view/444171.htm?func=retitle

**握手控制协议 Handshake**

主要设置控制串口的方式，软件控制，硬件控制，等等
MSDN 讲解http://msdn.microsoft.com/zh-cn/library/system.io.ports.handshake.aspx
串口相关内容转自[Tony.Y](http://www.cnblogs.com/tony-yang/archive/2009/06/03/learnserialport.html)


串口通信通过模拟串口简单做了一个实现，使用之前需添加允许using System.IO.Ports。
实例如下代码所示：
```
		private SerialPort sp=null;
 		/// <summary>
        /// 初始化串口并打开串口
        /// </summary>
        ///  <param name="protName">串口号</param>
        /// <param name="baudRate">波特率</param>
        /// <param name="DataBits">数据位</param>
        /// <param name="StopBits">停止位</param>
        /// <returns></returns>
        public bool OpenCom(string protName, int baudRate)
        {
            bool flag = true;
            if (sp == null)
            {
                sp = new SerialPort();
            }
            sp.PortName = protName;//串口号
            sp.BaudRate = baudRate;//波特率
            sp.DataBits = 8;
            sp.StopBits = StopBits.One;
            try
            {
                if (!sp.IsOpen)
                {
                    sp.Open();

                }
            }
            catch (Exception)
            {
                flag = false;
            }
            return flag;
        }
        /// <summary>
        /// 关闭串口
        /// </summary>
        /// <returns></returns>
        public void CloseCom()
        {
                if (sp.IsOpen)
                {
                    sp.Close();
                }
            
        }
            OpenCom("COM1", 9600);
            sp.WriteLine(data_opc);//需写入的数据
            CloseCom();
```
模拟串口可以实现，实操可能还需改动。目前先记录到此。
---
title: java spring学习总结（二）IoC Service Provider
date: 2008-05-12 15:35:00
tags: 
- java
- spring
categories: 总结
---

IoC Service Provider其实是一种抽象的概念，它可以指代任何将IoC场景中的业务对象绑定到一起的实现方式。当需要绑定的业务对象很多时，采取的方式需要一同改变。目前来看，有许多开源的产品通过各种方式为我们做了这部分的工作，Spring的IoC容器就是提供依赖注入服务的IoC Service Provider。

<!-- more -->

## IoC Service Provider的职责

IoC Service Provider的职责主要有两个，业务对象构建管理和业务对象间的依赖绑定。在Ioc场景中，业务对象无需唤醒所依赖的对象如何构建如何获得，IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来。而业务对象间的依赖绑定，是对于IoC Service Provider来说是其最终使命所在，需要保证每个业务对象在使用的时候，可以处于就绪的状态。

对于IoC Service Provider来说，需要知道被注入对象和依赖对象之间的关系。通常有直接编码方式，配置文件方式以及元数据方式。


Spring的IoC容器是一个提供IoC支持的轻量级容器，除了基本的IoC支持，它作为轻量级容器还提供了IoC之外的支持。如在Spring的IoC容器之上， Spring还提供了相应的AOP框架支持、企业级服务集成等服务。Spring提供了两种容器类型： BeanFactory和ApplicationContext。


BeanFactory是基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（lazy-load）。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景， BeanFactory是比较合适的IoC容器选择。



ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等。
ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说， ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。


## Spring 的IoC容器 之 BeanFactory

顾名思义，这就是生产Bean的工厂。
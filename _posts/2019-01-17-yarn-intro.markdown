---
layout:      post
title:       "Yarn Architecture"
subtitle:    " \"Yarn Architecture\""
date:        2019-01-17 19:27:38
author:      "Ming"
header-img:  "img/post-bg-universe.jpg"
tags:
    - yarn
    - hadoop
    - BigData
---

> “It's probably hard for you to imagine how it feels to have your destiny be predetermined at birth. ”


### Yarn的体系结构和运行原理
：运行计算框架(MapReduce, Storm, Spark, Flink等)的容器
#### 1. 主节点：ResourceManager(整体角度)
职责：
- 接收任务请求
- 资源的分配和任务的调度

#### 2.从节点：NodeManager(整体角度)
职责：
- 从ResourceManager获取任务和资源
- 执行

#### 3.YARN的架构
YARN的架构还是经典的主从结构(master/slave)。从大体上看，YARN服务由一个ResourceManager(RM)和多个NodeManager(NM)构成，ResourceManager为主节点(master)，NodeManager为从节点(slave)。
![](https://ws1.sinaimg.cn/large/005CDUpdly1fz9nf295u0j30fl0ax3yt.jpg)

从Yarn的架构图来看,它主要由ResourceManager, NodeManager, ApplicationMaster, Container等以下几个组件构成。
- Container是Yarn对计算机资源的抽象,它其实就是一组CPU和内存资源，所有的应用都会运行在Container中
- ApplicationMaster是对运行在Yarn中某个应用的抽象，它其实就是某个类型应用的实例，ApplicationMaster是应用级别的，它的主要功能就是向ResourceManager(全局的)申请计算资源(Containers)并且和NodeManager交互来执行和监控具体的task
- Scheduler是ResourceManager专门进行资源管理的一个组件，负责分配NodeManager上的Container资源，NodeManager也会不断发送自己的Container使用情况给ResourceManager.

(1) **ResourceManager(RM)**

RM是一个全局的资源管理器，负责整个系统的资源管理和分配。RM有两个主要的组件：Scheduler(调度器)和ApplicationMaser(应用程序管理器)。
- Scheduler(调度器):负责根据容量，队列等的熟悉约束，向各种运行的应用程序分配资源。调度程序是纯调度器，它不执行监视或跟踪应用程序的状态。此外，由于应用程序故障或硬件故障，它不能保证重新启动失败的任务。调度程序根据应用程序的资源需求执行其调度功能; 它基于包含诸如内存，cpu，磁盘，网络等元素的资源容器的抽象概念。YARN 提供了多种直接可用的调度器,比如 FairScheduler 和 Capacity Scheduler等。
- ApplicationMaster(应用程序管理器): 负责接收客户端提交的job;判断启动该job的ApplicationMster所需的资源(Container);提供服务监控ApplicationMaster的状态，以便在失败是重启ApplicationMaster。每个应用程序的ApplicationMaster有责任从调度程序协商适当的资源容器，跟踪其状态并监视进度。


(2) **ApplicationMaster(AM)**

AM更详细的描述是：当用户提交一个应用程序时，需要提供一个用以跟踪和管理程序的AM(申请任务ID)，它负责向ResourceManager申请资源，并要求NodeManager启动可以占用一定资源的任务。

AM主要功能包括：
- 与RM调度器协商获取资源(用Container表示)
- 将得到的任务进一步分配给内部的任务
- 与NM通信以启动/停止任务
- 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务

(3) **NodeManger(NM)**

NM是每个节点上的资源和任务管理器，一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；另一方面，它接收并处理来自AM的Container启动/停止等各种请求。

NM功能比较单一，只是负责Container状态的维护，并定时向RM的Secheduler汇报。

(4) **Container**

Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘和网络等。当AM向RM申请资源时，RM为AM返回的资源便是Container表示的(Secheduler将这些资源封装成Container).Yarn会为每个任务分配一个Container,且该任务只能使用该Container中描述的资源。

#### 4.调度任务的过程

**整体角度下的任务调度过程**：

![yarn任务调度](https://ws1.sinaimg.cn/large/005CDUpdly1fz9m4avwjpj317a0j148q.jpg)

**详细的工作流程**

收集了两种对于工作流程的描述，下面一一介绍，以供参考。

##### 1.第一种工作流程介绍

当用户向Yarn中提交一个应用程序后，Yarn将分两个阶段运行该应用程序：第一个阶段是启动ApplicationMaster;第二个阶段是由ApplicationMaster创建应用程序，为它申请资源，并监控它的整个运行流程，直到运行完成。

![执行过程](https://ws1.sinaimg.cn/large/005CDUpdly1fz9pujvxezj30mb0hpq9v.jpg)

1. 客户端向ResourceManager提交应用并请求一个ApplicationMaster实例
2. ResourceManager找到可以运行一个Container的NodeManager,并在这个Container容器中启动ApplicationMaster实例
3. ApplicationMaster向ResourceManager进行注册，注册之后客户端就可以查询ResourceManager获得自己ApplicationMaster的详细信息，以后就可以和自己的ApplicationMaster直接交互
4. 在平常的操作过程中，ApplicationMaster根据resource-request协议向ResourceManager发送resource-request请求
5. 当Container被成功分配之后，ApplicationMaster通过向NodeManager发送container-launch-specification信息来启动Container,container-launch-specification信息包含了能够让Container和ApplicationMaster交流所需要的资料
6. 应用程序的代码在启动的Container中运行，并把运行的进度，状态等信息通过application-specific协议发送给ApplicationMaster
7. 在应用程序运行期间，提交应用的客户端主动和ApplicationMaster交流获得应用的运行状态、进度更新等信息，交流的协议也是application-specific协议
8. 一旦应用程序执行完成并且所有相关工作也已经完成，ApplicationMaster向ResourceManager取消注册然后关闭，用到所有的Container也归还给系统。

##### 2.第二种工作流程介绍(MapReduce作业为例)

![Yarn作业运行流程](https://ws1.sinaimg.cn/large/005CDUpdly1fz9rro8a1jj30fe0epdih.jpg)

Yarn的作业运行，主要由以下几个步骤组成：

(1) 作业提交

client调用job.waitForCompletion方法，向整个集群提交MapReduce任务(第1步).新的作业ID(应用ID)由ResourceManager分配(第2步).作业的client核实作业的输出，计算输入的split，将作业的资源(包含jar包，配置信息，split信息)拷贝给HDFS(第3步).最后，通过调用ResourceManager的submitApplication()来提交作业(第4步).

(2) 作业初始化

当ResourceManager收到submitApplication()请求后，就将该请求发给调度器(scheduler)，调度器分配container,然后ResourceManager在该container内启动ApplicatioManager进程，由NodeManager监控(第5步).

MapReduce作业的ApplicationManager是一个主类为MRAppMaster的JAVA应用。其通过创造一些bookkeeping对象来监控作业的进度，得到任务的进度和完成报告(第6步).然后通过分布式文件系统得到由客户端计算好的输入split(第7步).然后为每一个split创建一个map任务，根据mapreduce.job.reduces创建reduce任务对象。

(3) 任务分配

如果任务很小，ApplicationManager会选择在其自己的JVM中运行任务。

如果不是小job，那么ApplicationManager向ResourceManager请求container来运行所有的map和reduce任务(第8步).这些请求是通过心跳来传输的，包括每个map任务的数据位置，比如存放输入split的主机名和rack。调度器利用这些信息来调度任务，尽量将任务分配给存储数据的节点，或者分配给和存放split的节点相同机架的节点。

(4) 任务运行

当一个任务由ResourceManager的调度器分配给一个container后，ApplicationManager通过联系NodeManager来启动container(第9步).任务由一个主类为YarnChild的JAVA应用执行。在运行任务之前首先本地化任务所需要的资源。比如作业配置，JAR文件，以及分布式缓存的所有文件(第10步).最后，运行map或reduce任务(第11步).

YarnChild运行在一个专用的JVM中，但是YARN不支持JVM重用。

(5) 进度和状态更新

Yarn中的任务将其进度和状态(包括counter)返回给ApplicationManager,客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新，展示给用户.

(6) 作业完成

除了向ApplicationManager请求任务进度外，客户端每5分钟都会通过waitForComplete()来检查作业是否完成。作业完成之后，ApplicationManager和Container会清理工作状态，OutputCommiter的作业清理方法也会被调用。作业的信息会被作业历史服务器存储以备之后用户检查。

#### 参考博文：

[Hadoop基础教程-第5章 YARN：资源调度平台（5.1 YARN介绍)](https://blog.csdn.net/chengyuqiang/article/details/72615108)

[一张图读懂yarn](https://www.jianshu.com/p/5dcc81c4e1cb)

[初步掌握Yarn的架构及原理](https://www.cnblogs.com/cxzdy/p/5494929.html)

**Just Keep Going !**
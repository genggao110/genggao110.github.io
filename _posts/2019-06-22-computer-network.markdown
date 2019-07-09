---
layout:     post
title:      "computer-network"
subtitle:   " \"network knowledge\""
date:       2019-06-22 15:37:00
author:     "Hux"
catalog: true
header-img: "img/about-bg-walle.jpg"
tags:
    - Network
    - TCP/IP
---

> "If you want to live your whole life free from pain, you must become either a god or else a corpse. Consider other men's troubles and that will comfort yours."

[TOC]

### 1. 网络基础知识总结

#### 1.1 OSI参考模型

OSI参考模型将实际的分组通信协议整理并分成了易于理解的7个分层。

分层名称 | 功能 | 详细概述
--- | --- | ---
应用层 | 针对特定应用的协议 | 为应用程序提供服务病规定应用中通信相关的细节。其包括文件传输、电子邮件、远程登录等协议
表示层 | 设备固有数据格式和网络标准数据格式的转换 | 将应用处理的信息转换为适合网络传输的格式，或将来自下一层的数据转换成上一层能够处理的数据格式，负责数据格式的转换
会话层 | 通信管理，负责建立和断开通信连接(数据流动的逻辑通路) | 负责建立和断开通信连接，决定采用何种连接方式，以及数据的分割等数据传输相关的管理
传输层 | 管理两个节点之间的数据传输。负责可靠传输(确保数据被可靠地传输到目标地址) | 保证数据的可靠传输，只在通信双方节点上进行处理，而无需再路由器上处理。
网络层 | 地址管理与路由选择 | 将数据传输到目标地址。目标地址可以是多个网络通过路由器连接而成的某一个地址。因此这一层主要负责寻址和路由选择。
数据链路层 | 互连设备之间 | haha

### 2. Test


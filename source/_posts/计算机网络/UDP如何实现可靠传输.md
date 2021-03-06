---
title: UDP如何实现可靠传输
author: Marlowe
tags:
  - UDP
  - 可靠传输
categories: 计算机网络
abbrlink: 16317
date: 2021-05-08 15:24:14
---

<!--more-->

### 概述

UDP不属于连接协议，具有资源消耗少，处理速度快的优点，所以通常音频，视频和普通数据在传送时，**使用UDP较多，因为即使丢失少量的包，也不会对接受结果产生较大的影响。**

传输层无法保证数据的可靠传输，只能通过应用层来实现了。实现的方式可以参照tcp可靠性传输的方式，只是实现不在传输层，实现转移到了应用层。

最简单的方式是**在应用层模仿传输层TCP的可靠性传输**。下面不考虑拥塞处理，可靠UDP的简单设计。

* 1、添加seq/ack机制，确保数据发送到对端
* 2、添加发送和接收缓冲区，主要是用户超时重传。
* 3、添加超时重传机制。

**详细说明：** 送端发送数据时，生成一个随机seq=x，然后每一片按照数据大小分配seq。数据到达接收端后接收端放入缓存，并发送一个ack=x的包，表示对方已经收到了数据。发送端收到了ack包后，删除缓冲区对应的数据。时间到后，定时任务检查是否需要重传数据。

目前有如下开源程序利用udp实现了可靠的数据传输。分别为**RUDP、RTP、UDT**。


### 开源程序

#### RUDP（Reliable User Datagram Protocol）

**RUDP 提供一组数据服务质量增强机制，如拥塞控制的改进、重发机制及淡化服务器算法等，** 从而在包丢失和网络拥塞的情况下， RTP 客户机（实时位置）面前呈现的就是一个高质量的 RTP 流。在不干扰协议的实时特性的同时，可靠 UDP 的拥塞控制机制允许 TCP 方式下的流控制行为。

#### RTP（Real Time Protocol）

**RTP为数据提供了具有实时特征的端对端传送服务**，如在组播或单播网络服务下的交互式视频音频或模拟数据。

应用程序通常在 UDP 上运行 RTP 以便使用其多路结点和校验服务；这两种协议都提供了传输层协议的功能。但是 RTP 可以与其它适合的底层网络或传输协议一起使用。如果底层网络提供组播方式，那么 RTP 可以使用该组播表传输数据到多个目的地。

RTP 本身并没有提供按时发送机制或其它服务质量（QoS）保证，它依赖于底层服务去实现这一过程。 RTP 并不保证传送或防止无序传送，也不确定底层网络的可靠性。 RTP 实行有序传送， RTP 中的序列号允许接收方重组发送方的包序列，同时序列号也能用于决定适当的包位置，例如：在视频解码中，就不需要顺序解码。

#### UDT（UDP-based Data Transfer Protocol）

基于UDP的数据传输协议（UDP-basedData Transfer Protocol，简称UDT）是一种互联网数据传输协议。**UDT的主要目的是支持高速广域网上的海量数据传输**，而互联网上的标准数据传输协议TCP在高带宽长距离网络上性能很差。

顾名思义，UDT建于UDP之上，并引入新的拥塞控制和数据可靠性控制机制。UDT是面向连接的双向的应用层协议。它同时支持可靠的数据流传输和部分可靠的数据报传输。由于UDT完全在UDP上实现，它也可以应用在除了高速数据传输之外的其它应用领域，例如点到点技术（P2P），防火墙穿透，多媒体数据传输等等。

### 参考

[UDP如何实现可靠传输](https://www.jianshu.com/p/6c73a4585eba)


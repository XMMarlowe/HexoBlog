---
title: IO多路复用
author: Marlowe
tags: IO
categories: 操作系统
abbrlink: 36405
date: 2021-04-10 19:04:10
---
<!--more-->

### 举例现实场景来理解

我们试想一下这样的现实场景:

一个餐厅同时有100位客人到店，当然到店后第一件要做的事情就是点菜。但是问题来了，餐厅老板为了节约人力成本目前只有一位大堂服务员拿着唯一的一本菜单等待客人进行服务。

* **方法A** 无论有多少客人等待点餐，服务员都把仅有的一份菜单递给其中一位客人，然后站在客人身旁等待这个客人完成点菜过程。在记录客人点菜内容后，把点菜记录交给后堂厨师。然后是第二位客人。。。。然后是第三位客人。很明显，只有脑袋被门夹过的老板，才会这样设置服务流程。因为随后的80位客人，再等待超时后就会离店(还会给差评)。

* **方法B** 老板马上新雇佣99名服务员，同时印制99本新的菜单。每一名服务员手持一本菜单负责一位客人(关键不只在于服务员，还在于菜单。因为没有菜单客人也无法点菜)。在客人点完菜后，记录点菜内容交给后堂厨师(当然为了更高效，后堂厨师最好也有100名)。这样每一位客人享受的就是VIP服务咯，当然客人不会走，但是人力成本可是一个大头哦(亏死你)。

* **方法C** 当客人到店后，自己申请一本菜单。想好自己要点的才后，就呼叫服务员。服务员站在自己身边后记录客人的菜单内容。将菜单递给厨师的过程也要进行改进，并不是每一份菜单记录好以后，都要交给后堂厨师。服务员可以记录号多份菜单后，同时交给厨师就行了。那么这种方式，对于老板来说人力成本是最低的；对于客人来说，虽然不再享受VIP服务并且要进行一定的等待，但是这些都是可接受的；对于服务员来说，基本上她的时间都没有浪费，基本上被老板压杆了最后一滴油水。

到店情况: 并发量。到店情况不理想时，一个服务员一本菜单，当然是足够了。所以不同的老板在不同的场合下，将会灵活选择服务员和菜单的配置.

* 客人: 客户端请求
* 点餐内容: 客户端发送的实际数据
* 老板: 操作系统
* 人力成本: 系统资源
* 菜单: 文件状态描述符。操作系统对于一个进程能够同时持有的文件状态描述符的个数是有限制的，在linux系统中$ulimit -n查看这个限制值，当然也是可以(并且应该)进行内核参数调整的。
* 服务员: 操作系统内核用于IO操作的线程(内核线程)
* 厨师: 应用程序线程(当然厨房就是应用程序进程咯)
* 餐单传递方式: 包括了阻塞式和非阻塞式两种。
    * 方法A: 阻塞式/非阻塞式 同步IO
    * 方法B: 使用线程进行处理的 阻塞式/非阻塞式 同步IO
    * 方法C: 阻塞式/非阻塞式 多路复用IO

### 多路复用IO实现
目前流程的多路复用IO实现主要包括四种: select、poll、epoll、kqueue。

| IO模型| 	相对性能	| 关键思路	| 操作系统	| JAVA支持情况| 
| :---:| :---:| :---:| :---:| :---| 
| select	| 较高	| Reactor	| windows/Linux	| 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型
| poll	| 较高	| Reactor	| Linux	| Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式
| epoll	| 高	| Reactor/Proactor	| Linux	| Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO
| kqueue	| 高	| Proactor	| Linux	| 目前JAVA的版本不支持

多路复用IO技术最适用的是“高并发”场景，所谓高并发是指1毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用IO技术发挥不出来它的优势。另一方面，使用JAVA NIO进行功能实现，相对于传统的Socket套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择。


### IO多路复用工作模式
epoll 的描述符事件有两种触发模式: LT(level trigger)和 ET(edge trigger)。

#### LT 模式
当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

#### ET 模式

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。


### 应用场景
很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

#### select 应用场景
select 的 timeout 参数精度为 1ns，而 poll 和 epoll 为 1ms，因此 select 更加适用于实时要求更高的场景，比如核反应堆的控制。 select 可移植性更好，几乎被所有主流平台所支持。

#### poll 应用场景
poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且epoll 的描述符存储在内核，不容易调试。
#### epoll 应用场景

只需要运行在 Linux 平台上，并且有非常大量的描述符需要同时轮询，而且这些连接最好是长连接。

#### select、poll、epoll有什么区别？

他们是NIO中多路复用的三种实现机制，是由Linux操作系统提供的。

用户空间和内核空间:操作系统为了保护系统安全，将内核划分为两个部分，一个是用户空间，一个是内核空间。用户空间不能直接访问底层的硬件设备，必须通过内核空间。

文件描述符File Descriptor(FD):是一个抽象的概念，形式上是一个整数，实际上是一个索引值。指向内核中为每个进程维护进程所打开的文件的记录表。当程序打开一个文件或者创建一个文件时，内核就会向进程返回一个FD。Unix,Linux

**select机制:** 会维护一个FD的结合 fd_set。将fd_set从用户空间复制到内核空间，激活socket。x64 2048(数组大小) fd_set是一个数组结构。

**Poll机制:** 和selecter机制是差不多的， 把fd_ set结构进行了优化，FD集合的大小就突破了操作系统的限制。pollfd结构来代替fd_set, 通过链表实现的。

**EPoll:** Event Poll.Epoll不再扫描所有的FD，只将用户关心的FD的事件存放到内核的一一个事件表当中。

**简单总结**

| |操作方式| 底层实现 | 最大连接数 | IO效率
|:---:|:---:|:---:|:---:|:---:|
|select| 遍历| 数组| 受限于内核| 一般|
|poll| 遍历| 链表| 无上限| 一般|
|epoll| 事件回调| 红黑树| 无上限| 高|

Java的NIO当中使用的是那种机制？

可以查看 DefaultSelectorProvider源码。
在windows 下，WindowsSelectorProvider。
而Linux下，根据Linux的内核版本，2.6版本以上，就是EPollSelectorProvider, 否则就是 默认的PollSelectorProvider。

### epoll详解文章推荐

[彻底搞懂epoll高效运行的原理](https://www.jianshu.com/p/31cdfd6f5a48)

[十个问题理解Linux epoll工作原理](https://cloud.tencent.com/developer/article/1831360)

[epoll 或者 kqueue 的原理是什么？](https://www.zhihu.com/question/20122137)

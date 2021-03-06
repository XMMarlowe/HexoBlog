---
title: Redis 持久化
author: Marlowe
tags: redis
categories: NoSQL
abbrlink: 15405
date: 2020-12-24 11:28:46
---
Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失，所以Redis提供了持久化功能！
<!--more-->
### RDB(Redis DataBase)
#### 简介

**在指定的时间间隔内将内存中的数据集写入磁盘，也就是行话讲的Snapshot快照，它恢复时时将快照文件直接读到内存里。** **Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。** 这就确保了极高的性能。如果需要进行大规模数据的恢复，且对数据恢复的完整性不是非常敏感，那么RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置！

在生产环境我们会将这个文件进行备份！

rdb保存的文件是dump.rdb  都是在配置文件中 快照中进行配置的！

#### 触发机制
1. save的规则满足情况下，会自动触发rdb规则
2. 执行flushall命令，也会触发我们的rdb规则！
3. 退出redis，也会产生rdb文件
备份就自动生成一个dump.rdb

#### 如何恢复rdb文件？

1. 只需将rdb文件放在redis启动目录就可以，redis启动的时候就会自动检查dump.rdb，并恢复其中的数据。
2. 查看需要存放的位置
```bash
127.0.0.1:6379> config get dir
1) "dir"
2) "D:\\Program Files\\redis-2.8.9"
```

#### 优点
适合大规模的数据恢复！

对数据的完整性要求不高

1. 一旦采用该方式，那么你的整个Redis数据库将只包含一个文件，这对于文件备份而言是非常完美的。比如，你可能打算每个小时归档一次最近24小时的数据，同时还要每天归档一次最近30天的数据。通过这样的备份策略，一旦系统出现灾难性故障，我们可以非常容易的进行恢复。

2. 对于灾难恢复而言，RDB是非常不错的选择。因为我们可以非常轻松的将一个单独的文件压缩后再转移到其它存储介质上。

3. 性能最大化。对于Redis的服务进程而言，在开始持久化时，它唯一需要做的只是fork出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行IO操作了。

4. 相比于AOF机制，如果数据集很大，RDB的启动效率会更高。

#### 缺点
需要一定的时间间隔进行操作，如果redis意外宕机了，这个最后一次修改的数据就没有了！

fork进程的时候，会占用一定的内存空间！

1. 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么RDB将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

2. 由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。

#### RDB持久化流程

![20210506150650](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210506150650.png)

1. Redis父进程首先判断：当前是否在执行save，或bgsave/bgrewriteaof（后面会详细介绍该命令）的子进程，如果在执行则bgsave命令直接返回。bgsave/bgrewriteaof 的子进程不能同时执行，主要是基于性能方面的考虑：两个并发的子进程同时执行大量的磁盘写操作，可能引起严重的性能问题。
2. 父进程执行fork操作创建子进程，这个过程中父进程是阻塞的，Redis不能执行来自客户端的任何命令
3. 父进程fork后，bgsave命令返回”Background saving started”信息并不再阻塞父进程，并可以响应其他命令
4. 子进程创建RDB文件，根据父进程内存快照生成临时快照文件，完成后对原有文件进行原子替换
5. 子进程发送信号给父进程表示完成，父进程更新统计信息

### AOF(Append Only File)

#### 简介
将我们的所有命令都记录下来，history，恢复的时候就把这个文件全部再执行一遍！
以日志的形式来记录每个读写操作，将Redis执行过的所有指令记录下来(读操作不记录),只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据,换言之，redis重启的话就根据日志文件的内容将写指令从前往后执行一次以完成数据的恢复工作

Aof保存的是appendonly.aof文件

默认是不开启的，需要手动进行配置，只需要将appendonly改为yes就可以开启aof！

重启redis就可以生效
如果这个aof文件有错，redis将无法启动，需要修复这个aof文件
redis提供了一个工具 `redis-check-aof --fix`
如果文件正常，重启就可以直接恢复了！

#### 重写规则说明
aof默认就是文件的无限追加，文件会越来越大!

如果aof文件大于64m，太大了，会fork一个新的进程来将文件重写！
 
#### 优点
每一次修改都同步，文件的完整性会更加好！

每秒同步一次，可能会丢失一秒的数据。

从不同步，效率最高。

1. 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。至于无同步，无需多言，我想大家都能正确的理解它。

2. 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。然而如果我们本次操作只是写入了一半数据就出现了系统崩溃问题，不用担心，在Redis下一次启动之前，我们可以通过redis-check-aof工具来帮助我们解决数据一致性的问题。

3. 如果日志过大，Redis可以自动启用rewrite机制。即Redis以append模式不断的将修改数据写入到老的磁盘文件中，同时Redis还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

4. AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

#### 缺点
相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢。

Aof运行效率比rdb慢，所以redis默认的配置就是rdb持久化。

1. 对于相同数量的数据集而言，AOF文件通常要大于RDB文件。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

2. 根据同步策略的不同，AOF在运行效率上往往会慢于RDB。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和RDB一样高效。

#### AOF持久化流程

![20210506150335](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210506150335.png)

1. 客户端发出 bgrewriteaof命令。

2. redis主进程fork子进程。

3. 父进程继续处理client请求，除了把写命令写入到原来的aof文件中。同时把收到的写命令缓存到 AOF重写缓冲区。这样就能保证如果子进程重写失败的话并不会出问题。

4. 子进程根据内存快照，按照命令合并规则写入到新AOF文件中。

5. 当子进程把内存快照写入临时文件中后，子进程发信号通知父进程。然后父进程把缓存的写命令也写入到临时文件。

6. 现在父进程可以使用临时文件替换老的aof文件，并重命名，后面收到的写命令也开始往新的aof文件中追加。


### 常用配置

#### RDB持久化配置

Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率，在打开6379.conf文件之后，我们搜索save，可以看到下面的配置信息：

```bash
save 900 1              #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。

save 300 10            #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。
```
#### AOF持久化配置

在Redis的配置文件中存在三种同步方式，它们分别是：
```bash
appendfsync always     #每次有数据修改发生时都会写入AOF文件。

appendfsync everysec  #每秒钟同步一次，该策略为AOF的缺省策略。

appendfsync no          #从不同步。高效但是数据不会被持久化。
```
### 如何选择
二者选择的标准，就是看系统是愿意牺牲一些性能，换取更高的缓存一致性（aof），还是愿意写操作频繁的时候，不启用备份来换取更高的性能，待手动运行save的时候，再做备份（rdb）。rdb这个就更有些 eventually consistent的意思了。

|命令|RDB|AOF|
|:---:|:---:|:---:|
|启动优先级|低|高|
|体积|小|大|
|恢复速度|块|慢|
|数据安全性|丢数据|根据策略决定|
|轻重|重|轻|

### 一些问题

#### Redis 持久化 之 AOF 和 RDB 同时开启，Redis听谁的？
听AOF的，RDB与AOF同时开启 默认无脑加载AOF的配置文件
相同数据集，AOF文件要远大于RDB文件，恢复速度慢于RDB
AOF运行效率慢于RDB,但是同步策略效率好，不同步效率和RDB相同

### 参考
[redis持久化的几种方式](https://www.cnblogs.com/chenliangcl/p/7240350.html)

[如何彻底理解Redis持久化？触发机制注意的点 | AOF持久化过程。](https://zhuanlan.zhihu.com/p/139726374)








---
title: ES写数据以及文档读写原理
author: Marlowe
tags: ES
categories: 中间件
abbrlink: 962
date: 2021-05-19 22:34:02
---

es写入数据的工作原理是什么啊？es查询数据的工作原理是什么？底层的lucence介绍一下呗？倒排索引了解吗？

<!--more-->

### ES写数据过程

1、客户端选择一个node发送请求过去，这个node就是coordinating node（协调节点）

2、coordinating node 对document进行路由，将请求转发给对应的node（有primary shard）

3、实际的node上的primary shard 处理请求，然后将数据同步到replica node。

4、coordinating node如果发现 primary node和所有replica node都搞定之后，就返回响应结果给客户端。

![20210519223812](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210519223812.png)


### ES读数据过程

可以通过doc id 来查询，会根据doc id进行hash，判断出来当时把doc id分配到了哪个shard上面去，从那个shard去查询。

1、客户端发送请求到任意一个node，成为coordinate node

2、coordinate node 对doc id进行哈希路由，将请求转发到对应node，此时会使用round-robin随机轮询算法，在primary shard 以及其所有replica中随机选择一个，让读请求负载均衡。

3、接收请求的node返回document给coordinate node。

4、coordinate node返回document给客户端。

### ES搜索数据过程

**ES最强大的是做全文检索。**

1、客户端发送请求到一个coordinate node。

2、协调节点将搜索请求转发到所有的shard对应的primary shard 或 replica shard ，都可以。

3、query phase：每个shard将自己的搜索结果（其实就是一些doc id）返回给协调节点，由协调节点进行数据的合并、排序、分页等操作，产出最终结果。

4、fetch phase：接着由协调节点根据doc id去各个节点上拉取实际的document数据，最终返回给客户端。

写请求是写入primary shard，然后同步给所有的replica shard

读请求可以从primary shard 或者 replica shard 读取，采用的是随机轮询算法。

### 写数据底层原理

![20210519224007](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210519224007.png)

1、先写入内存buffer，在buffer里的时候数据是搜索不到的；同时将数据写入translog日志文件。

如果buffer快满了，或者到一定时间，就会将内存buffer数据refresh 到一个新的segment file中，但是此时数据不是直接进入segment file磁盘文件，而是先进入 os cache。这个过程就是 refresh。

每隔1秒钟，es将buffer中的数据写入一个新的segment file，每秒钟会写入一个新的segment file，这个segment file中就存储最近1秒内 buffer中写入的数据。

2、但是如果buffer里面此时没有数据，那当然不会执行refresh操作，如果buffer里面有数据，默认1秒钟执行一次refresh操作，刷入一个新的segment file中。

操作系统里面，磁盘文件其实都有一个东西，叫做os cache，即操作系统缓存，就是说数据写入磁盘文件之前，会先进入os cache，先进入操作系统级别的

一个内存缓存中去。只要buffer中的数据被refresh 操作刷入os cache中，这个数据就可以被搜索到了。

**3、为什么叫es是准实时的？** 

NRT，全称 near real-time。默认是每隔1秒refresh一次的，所以es是准实时的，因为写入的数据1s之后才能被看到。

可以通过es的restful api或者 java api，手动执行一次 refresh操作，就是手动将buffer中的数据刷入os cache中，让数据立马就可以被搜索到。只要

数据被输入os cache中，buffer 就会被清空了，因为不需要保留buffer了，数据在translog里面已经持久化到磁盘去一份了。

4、重复上面的步骤，新的数据不断进入buffer和translog，不断将buffer数据写入一个又一个新的segment file中去，每次refresh完buffer清空，translog保留。

随着这个过程的推进，translog会变得越来越大。当translog达到一定长度的时候，就会触发commit操作。

5、commit操作发生的第一步，就是将buffer中现有的数据refresh到os cache中去，清空buffer。然后将一个commit point写入磁盘文件，里面标识者这个commit point 对应的所有segment file，同时强行将os cache中目前所有的数据都fsync到磁盘文件中去。最后清空现有 translog日志文件，重启一个translog，此时commit操作完成。

6、这个commit操作叫做flush。默认30分钟自动执行一次flush，但如果translog过大，也会触发flush。flush操作就对应着commit的全过程，我们可以通过es api，手动执行flush操作，手动将os cache中数据fsync强刷到磁盘上去。

7、**translog日志文件的作用是什么？**

执行commit 操作之前，数据要么是停留在buffer中，要么是停留在os cache中，无论是buffer 还是os cache都是内存，一旦这台机器死了，内存中的数据就全丢了。

所以需要将数据对应的操作写入一个专门的日志文件translog中，一旦此时机器宕机了，再次重启的时候，es会自动读取translog日志文件中的数据，恢复到内存buffer和os cache中去。

8、translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认情况下，可能有5s的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢5秒的数据。
也可以将translog设置成每次写操作必须是直接fsync到磁盘，但是性能会差很多。

9、es第一是准实时的，数据写入1秒后就可以搜索到：可能会丢失数据的。有5秒的数据，停留在buffer、translog os cache 、segment file os cache中，而不在磁盘上，

此时如果宕机，会导致5秒的数据丢失。

**10、总结：** 数据先写入内存buffer，然后每隔1s，将数据refresh到 os cache，到了 os cache数据就能被搜索到（所以我们才说es从写入到能被搜索到，中间有1s的延迟）。

每隔5s，将数据写入到translog文件（这样如果机器宕机，内存数据全没，最多会有5s的数据丢失），translog达到一定程度，或者默认每隔30min，会触发commit操作，将缓冲区的

数据都flush到segment file磁盘文件中。

数据写入 segment file之后，同时就建立好了倒排索引。

### 删除/更新数据底层原理

如果是删除操作，commit的时候会生成一个 .del文件，里面将某个doc标识为 deleted状态，那么搜索的时候根据 .del文件就知道这个doc是否被删除了。

如果是更新操作，就是将原来的doc标识为deleted状态，然后重新写入一条数据。

buffer 每refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，这样下来segment file会越来越多，此时会定期执行merge。

每次merge的时候，会将多个segment file合并成一个，同时这里会将标识为 deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个

commit point，标识所有新的 segment file，然后打开segment file供搜索使用，同时删除旧的segment file。

### 文档读写模型实现原理

#### 简介

ElasticSearch，每个索引被分成多个分片（默认每个索引5个主分片primary shard），每个分片又可以有多个副本。当一个文档被添加或删除时（主分片中新增或删除），其对应的复制分片之间必须保持同步。那如何保持分片副本同步呢？这就是本篇重点要阐述的，即数据复制模型。

ElasticSearch的数据复制模型是基于主从备份模型的。每一个复制组中会有一个主分片，其他分片均为复制分片。主分片服务器是所有索引操作的主要入口点（索引、更新、删除操作）。一旦一个索引操作被主服务器接受之后主分片服务器会将其数据复制到其他副本。


#### 基本写模型

ElasticSearch每个索引操作首先会进行路由选择定位到一个复制组，默认基于文档ID(routing)，其基本算法为hash(routing) % (primary count)。一旦确定了复制组，则该操作将被转发到该组的主分片（primary shard）。主分片服务器负责验证操作并将其转发到其他副本。

由于副本可以离线（一个复制组并不是要求所有复制分片都在线才能运作），可能不需要复制到所有副本。ElasticSearch会维护一个当前在线的副本服务器列表，这个列表被称为in-sync副本，由主节点维护。也就是当主分片接收一个文档后，需要将该文档复制到in-sync列表中的每一台服务器。

##### 主分片的处理流程如下：

验证请求是否符合Elasticsearch的接口规范，如果不符合，直接拒绝。
在主分片上执行操作(例如索引、更新或删除一个文档)。如果执行过程中出错，直接返回错误。
将操作转发到当前同步副本集的每个副本。如果有多个副本，则并行执行。（in-sync当前可用、激活的副本）。
一旦所有的副本成功地执行了操作并对主服务器进行了响应，主服务器向客户端返回成功。

写请求的流程如下图所示（图片来源于《Elasticsearch权威指南》，如有侵权，马上删除）：

![20210519224501](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210519224501.png)

##### 异常处理机制：

在索引过程中，许多情况会引发异常，例如磁盘可能会被破坏、节点之间网络彼此断开，或者一些配置错误可能导致一个副本的操作失败，尽管它在主服务器上是成功的。上述原因虽然很少会发生，但我们在设计时还是必须考虑如果发生错误了该如何处理。

另外一种情况异常情况也不得不考虑。如果主服务器不可用，ES集群该如何处理呢？

此时会触发复制组内的主服务器选举，选举后新的主节点会向master服务器发送一条消息，然后，该请求会将被转发到新的主服务器进行处理。此过程该请求在主节点选主期间会阻塞（等待），默认情况下最多等待1分钟。

注：主服务器(master)会监控各个节点的健康状况，并可能决定主动降级主节点。这通常发生在持有主节点的节点通过网络问题与集群隔离的情况下。

为了更好的理解master服务器与主分片所在服务器的关系，下面给出一个ElasticSearch的集群说明图：（图片来源于《Elasticsearch权威指南》，如有侵权，马上删除）

![20210519224525](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210519224525.png)

其中NODE1为整个集群的master服务器，而第一个复制组(P0,R0,RO,其主分片所在服务器NODE3)，第二个复制组(P1,R1,R1,其主分片所在服务器NODE1)。

一旦在主服务器上成功执行了操作，主服务器就必须确保数据最终一致，即使由于在副本上执行失败或由于网络问题导致操作无法到达副本（或阻止副本响应）造成的。

为了避免数据在复制组内数据的不一致性（例如在主分片中执行成功，但在其中一两个复制分片中执行失败），主分片如果未在指定时间内（默认一分钟）未收到复制分片的成功响应或是收到错误响应，主分片会向Master服务器发送一个请求，请求从同步副本中删除有问题的分片，最终当主分片服务器确向Master服务器的确认要删除有问题的副本时，Master会指示删除有问题的副本。同时，master还会指示另一个节点开始构建新的分片副本，以便将系统恢复到一个健康状态。

主分片将一个操作转发到副本时，首先会使用副本数来验证它仍然是活动的主节点。如果由于网络分区（或长GC）而被隔离，那么在意识到它已经被降级之前，它可能会继续处理传入的索引操作并转发到从服务器。来自陈旧的主服务器的操作将会被副本拒绝。当主接受到来自副本的响应为拒绝它的请求时，此时的主分片会向Master服务器发送请求，最终将知道它已经被替换了，后续操作将会路由到新的主分片服务器上。

如果没有副本，那会发生什么呢？

这是一个有效的场景，可能由于配置而发生，或者是因为所有的副本都失败了。在这种情况下，主分片要在没有任何外部验证的情况下处理操作，这可能看起来有问题。另一方面，主分片服务器不能自己失败其他的分片（副本），而是请求master服务器代表它这样做。这意味着master服务器知道主分片是该复制组唯一的可用拷贝。因此，我们保证master不会将任何其他（过时的）分片副本提升为一个新的主分片，并且任何索引到主分片服务器的操作都不会丢失。当然这样意味着我们只使用单一的数据副本，物理硬件问题可能导致数据丢失。请参阅Wait For Active Shards，以获得一些缓解选项,(该参数项将在下一节中详细描述)。

注：在一个ElasticSearch集群中，存在两个维度的选主。Master节点的选主、各个复制组主分片的选主。

#### 基本读模型

在Elasticsearch中，可以通过ID进行非常轻量级的查找，也可以使用复杂的聚合来获取非平凡的CPU能力。主备份模型的优点之一是它使所有的分片副本保持相同（除了异常情况恢复中）。通常，一个副本就足以满足读取请求。

当一个节点接收到read请求时，该节点根据路由规则负责将其转发给相应的数据节点，对响应进行整理，并对客户端作出响应。我们称该节点为该请求的协调节点。基本流程如下：

将读请求路由到到相关的分片节点。注意，由于大多数搜索条件中不包含分片字段，所以它们通常需要从多个分片组中读取数据，每个分片代表一个不同的数据子集（默认5个数据子集，因为ElasticSearch默认的主分片个数为5个）。
从每个分片复制组中选择一个副本。读请求可以是复制组中的主分片，也可以是其副本分片。在默认情况下，ElasticSearch分片组内的读请求负载算法为轮询。
根据第二步选择的各个分片，向选中的分片发送请求。
汇聚各个分片节点返回的数据，然后返回个客户端，注意，如果带有分片字段的查询，将之后转发给一个节点，该步骤可省略。
异常处理：
当一个碎片不能响应一个read请求时，协调节点将从同一个复制组中选择另一个副本，并向其发送查询请求。重复的失败会导致没有分片副本可用。在某些情况下，比如搜索，ElasticSearch会更倾向于快速响应（失败后不重试），返回成功的分片数据给客户端 ，并在响应包中指明哪些分片节点发生了错误。


#### Elasticsearch主备模型隐含含义

在正常操作下，每个读取操作一次为每个相关的复制组执行一次。只有在失败条件下，同一个复制组的多个副本执行相同的搜索。
由于数据首先是在主分片上进行索引后，然后才转发请求到副本，在转发之前数据已经在主分片上发生了变化，所以在并发读时，如果读请求被转发到主分片节点上，那该数据在它被确认之前（主分片再等待所有副本全部执行成功）就已经看到了变化。【有点类似于数据库的读未提交】。
主备模型的容错能力为两个分片（1主分片，1副本）

#### ElasticSearch 读写模型异常时可造成的影响

在失败的情况下，以下是可能的：

1个分片节点可能减慢整个集群的索引性能
因为在每次操作期间(索引)，主分片在本地成功执行索引动作后，会转发请求到期复制分片节点上，此时主分片需要等待所有同步副本节点的响应，单个慢分片可以减慢整个复制组的速度。当然，一个缓慢的分片也会减慢那些被路由到它的搜索。
脏读
一个孤立的主服务器可以公开不被承认的写入。这是由于一个孤立的主节点只会意识到它在向副本发送请求或向主人发送请求时被隔离。在这一点上，操作已经被索引到主节点，并且可以通过并发读取读取。Elasticsearch可以通过在每秒钟（默认情况下）对master进行ping来减少这种风险，并且如果没有已知的主节点，则拒绝索引操作。
本文详细介绍了ElasticSearch文档的读写模型的设计思路，涉及到写模型及其异常处理、读模型及其异常处理、主备负载模型背后隐含的设计缺陷与ElasticSearch在异常情况带来的影响。


### 参考

[ES读写数据的工作原理以及文档读写模型实现原理](https://blog.csdn.net/zhutianrong520/article/details/103445179)

Redis简介
===

Redis是一个开源的基于BSD协议的内存 **富数据结构存储** ，可用做数据库、缓存及消息中间件。
支持的数据结构有 [字符串](/topics/data-types-intro#strings) 、 [哈希](/topics/data-types-intro#hashes) 、 [列表](/topics/data-types-intro#lists) 、 [集合](/topics/data-types-intro#sets) 、支持范围查询的 [有序集合](/topics/data-types-intro#sorted-sets) 、 [位图](/topics/data-types-intro#bitmaps) 、 [hyperloglogs](/topics/data-types-intro#hyperloglogs) 和 支持半径查询的 [地理位置索引](/commands/geoadd) 。
Redis内置了 [复制](/topics/replication) 、 [Lua脚本](/commands/eval) 、 [LRU淘汰](/topics/lru-cache) 、 [事务](/topics/transactions) 、 不同级别的 [磁盘持久化](/topics/persistence) 、 保证高可用性的 [Redis Sentinel](/topics/sentinel) 以及自动分区的 [Redis集群](/topics/cluster-tutorial) 。

你可以对这些类型执行 **原子操作** ，例如 [字符串追加](/commands/append) ； [自增哈希表中的值](/commands/hincrby) ；
[向列表中推入元素](/commands/lpush) ； [计算集合的交集](/commands/sinter) ，
[并集](/commands/sunion) 和 [差集](/commands/sdiff) ；
或 [取得有序集合中排名在最前面的成员](/commands/zrangebyscore) 。

为了达到杰出的性能，Redis以 **内存数据集** 的方式进行工作。
根据不同的用例，可以选择两种持久化方式：每隔一段时间 [将数据集转储到硬盘上](/topics/persistence#snapshotting) ，或 [将每条命令追加到日志里](/topics/persistence#append-only-file) 。
如果只是需要多功能、网络化的内存缓存，可以选择禁用持久化。

Redis也支持详细的 [主从复制](/topics/replication)设置，具有同步快速且非阻塞优先、网络失败自动重连。

还有其他一些特性包括：
* [事务](/topics/transactions)
* [发布/订阅](/topics/pubsub)
* [Lua脚本](/commands/eval)
* [带有生存时间的键](/commands/expire)
* [LRU键淘汰制度](/topics/lru-cache)
* [自动故障转移](/topics/sentinel)

你可以在 [大多数编程语言](/clients) 中使用Redis。

Redis使用 **ANSI C** 编写，可以在多数POSIX系统如Linux、\*BSD 及 OS X中运行并且不需要外部依赖。
Linux和OS X是Redis进行开发及多数测试的两个操作系统，我们 **推荐使用Linux部署** 。
Redis或许可以在Solaris的派生系统中运行如SmartOS，我们将会尽力提供支持。
官方没有提供对Windows的支持，但微软开发并维护了一个 [Win64版本的Redis](https://github.com/MSOpenTech/redis) 。

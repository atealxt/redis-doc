Redis数据类型及概念的简介
===

Redis不仅仅是一个简单键-值存储应用，事实上它是一个 *富数据类型的服务程序* ，支持多种类型的值。
意思就是说，在传统的键值存储应用中使用字符串键关联字符串值，而在Redis中值并不限于普通字符串，而可以是多种复杂的数据结构。
以下列出了所有Redis支持的数据结构，后面将一一进行说明：

* 二进制安全的字符串。
* 列表：保持插入顺序的字符串集合。简单来说就是 *链表* 。
* 集合：值唯一的无序的字符串集合。
* 有序集合：类似于集合，但是每个字符串元素与一个浮点数值进行关联，叫做 *分数* 。元素以分数进行排序，所以可以取得指定区间的部分元素（例如取得最开始或最后的10个元素），普通集合不支持。
* 哈希表，映射字段与值的关系。字段与值均为字符串。这个数据结构和Ruby或Python的哈希表非常相像。
* 比特数组（或简单的位图）：可以使用特别的命令像比特数组一样处理字符串：设置和清除单独的比特、计算比特1的数量、查找第一个比特1等等。
* HyperLogLogs：这是一个概率类数据结构，用来推算集合的势。不要被吓到，它比看起来要简单，后面会有介绍。

为了能从  [命令手册](/commands) 中找到解决问题的合适命令，有时候需要理解这些数据类型是如何工作的才能做出正确的选择。
此文档是Redis数据类型及常用模式的快速指南。

对于所有的例子，我们将使用一个简单而方便的命令行工具 `redis-cli` 向Redis服务器发送命令。

Redis的键
---

Redis的键是二进制安全的，这意味着可以使用任何二进制序列作为键，从字符串如 "foo" 到JPEG文件内容都可以。
空字符串也是合法的键。

关于键的其他一些规则：

* 使用过长的键并不是个好主意，比如1024字节。不只是因为消耗内存，在数据集中寻找键也可能需要对键进行数次比较，代价很大。
即使需要对很大的值进行匹配，使用哈希（例如SHA1）也会更好，特别是对于内存和带宽来说。
* 过短的键通常也不好。
  使用 "u1000flw" 作为键比使用 "user:1000:followers" 没有多大改进，而后者更具有可读性。相对于值来说键所占的空间很小，也不会产生多大的额外空间。
  但短的键确实能够减少一点内存消耗，重要的是找到平衡。
* 建议使用命名规范。
  例如 "object-type:id:field" 就是一个好主意，如 "user:1000:password" 。
  标点或横杠等符号经常被用来分隔多个词，比如 "comment:1234:reply.to" 或 "comment:1234:reply-to" 。
* 键的最大取值为512 MB。

<a name="strings"></a>
Redis字符串
---

Redis的字符串类型是Redis键能关联的最简单的值类型。
这是Memcached的唯一数据类型，Redis新手使用起来也很轻松。

Redis的键是字符串类型。当值也使用字符串时，实际上是将一个字符串映射到另一个字符串上。
字符串数据类型适用于很多场景，如缓存HTML代码片段或页面。

让我们从实际使用中来了解字符串类型，通过 `redis-cli` （本文中的所有的例子都将使用 `redis-cli` 运行）。

    > set mykey somevalue
    OK
    > get mykey
    "somevalue"

正如所见， `SET` 和 `GET` 是用来设置和取得字符串值的命令。
注意当键已经关联值时，无论原值是不是字符串类型， `SET` 都将会进行替换。
所以 `SET` 就是一个赋值过程。

值可以为任何种类的字符串（包括二进制数据），例如jpeg图片。值不能大于512MB。

 `SET` 有几个有意思的可选参数。
例如在键已经存在时让 `SET` 失败，或相反只在键已经存在时执行：

    > set mykey newval nx
    (nil)
    > set mykey newval xx
    OK

虽然字符串是Redis基本的值类型，但是可以执行一些有趣的操作。
例如原子性自增：

    > set counter 100
    OK
    > incr counter
    (integer) 101
    > incr counter
    (integer) 102
    > incrby counter 50
    (integer) 152

命令 [INCR](/commands/incr) 将字符串值解析为整数，并进行自增一，最后把结果转换成新值。
类似的命令有 [INCRBY](/commands/incrby) 、 [DECR](/commands/decr) 和 [DECRBY](/commands/decrby) 。
在系统内部它们调用相同的命令，只在调用方法上略有不同。

INCR的原子性是指，多个客户端同时对相同键执行INCR不会引起竞争。
例如不可能发生客户端1读取到 "10" ，客户端2同时读取到 "10" ，二者都自增到了11，并设置新值为11。
最终的值永远是12，读取-自增-设置 操作不可能有多个客户端同时执行。

Redis有很多操作字符串的命令。
例如命令 `GETSET` 为键设置一个新值并返回旧值。
可以在例如一个网站，每接收到一次访问的时候对Redis键进行自增的时候使用它。 
每小时收集一次信息，不能丢失一次自增。
对键使用 `GETSET` ，设置新值 "0" 同时读取旧值。

在一个命令里同时设置或读取多个键可以有效的降低延时。
它们是命令 `MSET` 和 `MGET` ：

    > mset a 10 b 20 c 30
    OK
    > mget a b c
    1) "10"
    2) "20"
    3) "30"

使用 `MGET` 时Redis返回一个值数组。

修改和查询键空间
---

有一些键操作命令无需指定键的类型。

例如命令 `EXISTS` 返回1或0表示键是否存在于数据集中，命令 `DEL` 删除键及关联值，不论值是什么。

    > set mykey hello
    OK
    > exists mykey
    (integer) 1
    > del mykey
    (integer) 1
    > exists mykey
    (integer) 0

从例子中可以看到 `DEL` 在删除键（键存在）时返回1，在键不存在（没有相同名称的键）时返回0。

有很多键空间相关的命令， `TYPE` 返回保存在指定键的值类型，经常和上面两个命令一起使用：

    > set mykey x
    OK
    > type mykey
    string
    > del mykey
    (integer) 1
    > type mykey
    none

Redis超时策略：设置键的生存时间
---

在介绍更复杂的数据结构之前，先来说说另一个与值类型无关的功能，叫做 **Redis 超时** 。
简单来讲就是给键设置超时，即有限的生存时间。
当过了生存时间后，键自动被消除，就像用户调用 `DEL` 删除了它一样。

这有几个关于Redis超时的特性：

* 精度可以使用秒或毫秒。
* 底层超时时间的最小单位是毫秒。
* 超时信息会被复制和持久化至磁盘，在Redis服务停止时也会生效（意味着Redis保存的是键的超时日期）。

设置超时很简单：

    > set key some-value
    OK
    > expire key 5
    (integer) 1
    > get key (immediately)
    "some-value"
    > get key (after some time)
    (nil)

当经过5秒钟后调用第二次 `GET` 时，上面的键就消失了。
例子使用的是 `EXPIRE` 给键设置超时（它也可以用来修改键的超时时间，而 `PERSIST` 可以用来删除超时设置让键持久存在）。
另外还可以用其他一些Redis命令在创建键的同时设置超时时间，例如在 `SET` 命令中添加选项：

    > set key 100 ex 10
    OK
    > ttl key
    (integer) 9

上面的例子给键设置字符串值 `100` ，并在十秒钟后过期。
之后调用 `TTL` 命令查看了键的剩余生存时间。

以毫秒为单位设置和检查超时，请见命令 `PEXPIRE` 、 `PTTL` ，及 `SET` 的完整选项列表。

<a name="lists"></a>
Redis列表
---

在讲解列表数据类型之前，先来说点理论，因为 *列表* 这个词在IT圈内常常被错误使用。
例如 "Python Lists" 并不如其名称提示的那样是链表，而是数组（相同的数据类型在Ruby中叫做Array）。

一个非常普通的列表就是一个元素序列：如10,20,1,2,3。
但是用数组实现的列表与用 *链表* 实现的列表具有非常大的差别。

Redis的列表是用链表实现的。
这意味着即使列表有上百万个元素，向其头或尾添加一个新元素消耗的时间为 *常量* 。
使用 `LPUSH` 命令向一个有十个元素的列表表头添加一个元素，与向一个有一千万个元素的列表表头添加一个元素的速度相同。

劣势是，根据 *索引* 访问元素在数组实现方式中非常迅速（常数时间查找），而在链表实现方式中没那么快（操作需要的工作量与索引访问的元素数成正比）。

Redis的列表使用链表实现是因为对于数据库系统来说，向很长的列表中添加元素必须要非常迅速。
另一个重要的优势是，也是下面所能看到的，Redis列表的开销可以控制在常数长度与时间内。

当需要在大量元素中快速访问时，可以使用另一种数据结构，叫做有序集合。有序集合将会在后面进行介绍。

Redis列表入门
---

命令 `LPUSH` 从左侧（头部）向列表添加新元素，命令 `RPUSH` 从右侧（尾部）向列表添加新元素。
最后命令 `LRANGE` 从列表中取得指定范围的元素：

    > rpush mylist A
    (integer) 1
    > rpush mylist B
    (integer) 2
    > lpush mylist first
    (integer) 3
    > lrange mylist 0 -1
    1) "first"
    2) "A"
    3) "B"

注意 [LRANGE](/commands/lrange) 接收两个参数，为返回的第一个及最后一个元素的索引。
两个参数都可以为负数，让Redis从末尾位置开始计数：即-1为列表的最后一个元素，-2为倒数第二个元素，以此类推。

正如所见 `RPUSH` 从列表右侧添加元素，而后的 `LPUSH` 从列表左侧添加元素。

两个命令的参数都是 *可变的* ，可以一次性向列表添加多个元素：

    > rpush mylist 1 2 3 4 5 "foo bar"
    (integer) 9
    > lrange mylist 0 -1
    1) "first"
    2) "A"
    3) "B"
    4) "1"
    5) "2"
    6) "3"
    7) "4"
    8) "5"
    9) "foo bar"

Redis列表的一个重要的功能就是 *移除元素* ，在获取元素的同时从列表中将其删除。
列表左右两侧都可以删除元素，正如添加元素一样：

    > rpush mylist a b c
    (integer) 3
    > rpop mylist
    "c"
    > rpop mylist
    "b"
    > rpop mylist
    "a"

上面的命令添加了三个元素并且移除了三个元素，所以最后列表为空，没有元素可供移除了。
如果尝试再移除一个元素，将得到结果：

    > rpop mylist
    (nil)

当列表没有可供移除的元素时返回一个NULL值。

列表的常见用例
---

列表适用于多种任务，下面是两个非常具有代表性的用例：

* 记住用户最近的社交网络更新。
* 使用生产者-消费者模式的进程间通信，生产者向列表添加元素，消费者（通常为 *工人* ）接收元素执行操作。Redis有一系列可靠而高效的列表命令。

例如流行的Ruby库 [resque](https://github.com/resque/resque) 和 [sidekiq](https://github.com/mperham/sidekiq) 就是在底层使用了Redis的列表实现后台任务。

流行的Twitter社交网络将 [用户最新发布的消息](http://www.infoq.com/presentations/Real-Time-Delivery-Twitter) 放入了Redis列表。

如果从最基础来说，就比如用来提升社交网络首页图片分享中最近发布照片的加载速度。

* 用户每次新发布一张照片，使用 `LPUSH` 将它的ID放入列表。
* 当用户访问首页时，使用 `LRANGE 0 9` 取得最新的十张照片。

限制长度的列表
---

很多时候使用列表只是想要保存 *最后若干个元素* ，比如：社交网络上的更新、日志以及其他。

Redis允许限制列表的长度，只记住最近的N个元素，使用命令 `LTRIM` 丢弃掉所有旧元素。

命令 `LTRIM` 与 `LRANGE` 类似，但不是  **显示指定区间的元素** 而是将指定区间的元素设置成为列表的新值。
区间外的所有元素将被移除。

看个例子会更清楚些：

    > rpush mylist 1 2 3 4 5
    (integer) 5
    > ltrim mylist 0 2
    OK
    > lrange mylist 0 -1
    1) "1"
    2) "2"
    3) "3"

上面的命令 `LTRIM` 告诉Redis只保留索引从0至2的元素，舍弃其他所有元素。
这能够实现一个很简单但很有用的功能：在保证一致性的同时将列表插入操作与截取操作放在一起执行，添加元素并舍弃超过界限的元素：

    LPUSH mylist <some element>
    LTRIM mylist 0 999

上面的列表在添加完新元素后只保留最新的1000个元素。
然后再使用 `LRANGE` 就可以只访问最新的元素而不保留旧数据。

注意： `LRANGE` 虽然是一个复杂度为 O(N) 的命令，但在访问列表头尾小范围区间的元素时则是定长时间操作。

列表的阻塞操作
---

列表的特殊之处可以让它很容易的实现队列，创建阻塞的进程通信系统：阻塞操作。

比如一个进程向列表里添加元素，另一个进程对列表中的元素进行处理。
通常这叫生产者/消费者模式，可以很容易的以下面的方式实现：

* 生产者调用 `LPUSH` 向列表中添加元素。
* 消费者调用 `RPOP` 从列表中提取并处理元素。

但是有时候没有待处理元素列表为空，这时 `RPOP` 仅仅返回一个NULL。
消费者被强迫等待一段时间并使用 `RPOP` 重试。
这被称为 *拉取* ，在这种场合下并不适用，存在几个缺点：

1. 让Redis和客户端不断执行无用的命令（当列表为空时所有请求都没有意义，只会返回NULL）。
2. 元素处理将会有延迟，因为当工作进程收到NULL后会进行等待。缩小再次调用 `RPOP` 的等待时间又会加剧问题1，例如更多无用的调用。

所以Redis提供了阻塞版本的 `RPOP` 和 `LPOP` 叫做 `BRPOP` 和 `BLPOP` 可以在列表为空时阻塞：只有当列表中有新元素时调用才会返回，或达到用户指定的超时时间。

这是一个工作进程调用 `BRPOP` 的例子：

    > brpop tasks 5
    1) "tasks"
    2) "do_something"

它的意思是："等待列表 `tasks` 中的元素，如果在5秒钟过后没有元素时也返回"。

可以使用0也就是不限制等待元素的超时时间，也可以同时等待多个列表，任一列表非空时都能接收到元素。

关于 `BRPOP` 的一些注意事项：

1. 客户端排序：第一个阻塞等待列表的客户端将会先得到元素，以此类推。
2. 返回值的格式不同于 `RPOP` ：返回值为两个元素的数组，包含键的名称。因为 `BRPOP` 和 `BLPOP` 可以同时阻塞等待多个列表。
3. 等待超时后返回NULL。

关于列表与阻塞操作还有一些需要知道的事，推荐阅读以下文章：

* 使用 `RPOPLPUSH` 可以创建可靠队列或反转队列。
* 还有一个阻塞操作命令，叫做 `BRPOPLPUSH` 。

键的自动创建与删除
---

截至目前的例子中没有哪个需要在添加元素前创建空列表，或在列表没有元素后删除空列表。
这个由Redis来管理。当列表为空时删除所在键，或在向键还不存在的列表添加元素比如 `LPUSH` 时创建空列表。

并不只是列表，所有Redis的多元素组合数据类型——集合，有序集合和哈希表。

基本上总结起来有三点：

1. 当向这些数据类型中添加元素时，如果目标键不存在，在添加元素前会创建一个空的数据类型。
2. 当从这些数据类型中删除元素后，如果数据类型的值为空，键会自动被销毁。
3. 在空的键上调用只读命令如 `LLEN` （返回列表的长度）或删除元素时，产生的结果永远如在空的数据类型上执行一样。

关于第1点的例子：

    > del mylist
    (integer) 1
    > lpush mylist 1 2 3
    (integer) 3

然而不能在错误类型的键上执行此操作：

    > set foo bar
    OK
    > lpush foo 1 2 3
    (error) WRONGTYPE Operation against a key holding the wrong kind of value
    > type foo
    string

关于第2点的例子：

    > lpush mylist 1 2 3
    (integer) 3
    > exists mylist
    (integer) 1
    > lpop mylist
    "3"
    > lpop mylist
    "2"
    > lpop mylist
    "1"
    > exists mylist
    (integer) 0 

在所有元素都弹出后键就不存在了。

关于第3点的例子：

    > del mylist
    (integer) 0
    > llen mylist
    (integer) 0
    > lpop mylist
    (nil)

<a name="hashes"></a>
Redis哈希
---

Redis的哈希正如其名字“哈希”那样，使用键值对：

    > hmset user:1000 username antirez birthyear 1977 verified 1
    OK
    > hget user:1000 username
    "antirez"
    > hget user:1000 birthyear
    "1977"
    > hgetall user:1000
    1) "username"
    2) "antirez"
    3) "birthyear"
    4) "1977"
    5) "verified"
    6) "1"

它就是一个键值对集合。
用哈希存取对象很方便，数量也基本上没有限制（除非内存不够），所以应用程序有很多种使用哈希的方式。

命令 `HMSET` 用来给哈希设置多个字段， `HGET` 则用来取得哈希的一个字段。 `HMGET` 和 `HGET` 类似，不同的是取得的值是一个数组：

    > hmget user:1000 username birthyear no-such-field
    1) "antirez"
    2) "1977"
    3) (nil)

还有一些可以操作单个字段的命令，比如 `HINCRBY` ：

    > hincrby user:1000 birthyear 10
    (integer) 1987
    > hincrby user:1000 birthyear 10
    (integer) 1997

关于哈希命令的完整列表请见命令[文档](http://redis.io/commands#hash)。

有一点值得提一下，小的哈希（比如少量值不大的元素）以特殊方式进行了编码，内存的使用效率非常高。

<a name="sets"></a>
Redis集合
---

Redis集合是二进制安全的字符串无序集。
可以使用命令 `SADD` 向集合中添加元素。
也可以使用其他命令测试元素是否存在、取得多个集合的交集、并集或差异等等。

    > sadd myset 1 2 3
    (integer) 3
    > smembers myset
    1. 3
    2. 1
    3. 2

程序向集合中添加了三个元素并让Redis返回它们。
可以看到它们并不是有序的——Redis对于每次调用的返回元素顺序都有可能不同。

有一些命令可以检测集合中的元素。例如检测元素是否存在：

    > sismember myset 3
    (integer) 1
    > sismember myset 30
    (integer) 0

"3" 是集合的成员， "30" 不是。

集合非常适合表达对象之间的联系。
例如可以使用集合轻松的实现标签功能。

一种简单的方法是为每个需要标记的对象建立一个集合。
集合包含标记的ID与关联对象。

比如标记新文章。
如果ID为1000的文章有标签1、2、5和77，那么就可以使用一个集合进行关联：

    > sadd news:1000:tags 1 2 5 77
    (integer) 4

也可以取得相对的关系：一个标签对应的所有新闻：

    > sadd tag:1:news 1000
    (integer) 1
    > sadd tag:2:news 1000
    (integer) 1
    > sadd tag:5:news 1000
    (integer) 1
    > sadd tag:77:news 1000
    (integer) 1

可以这样简单的取得对象的所有标签：

    > redis-cli smembers news:1000:tags
    1. 5
    2. 1
    3. 77
    4. 2

注意：例子中假设存在有另一个数据结构，例如Redis哈希，关联着标签ID和名称。

Redis还有一些可以实现别的操作的命令。
例如取得所有包含标签1、2、10和27的列表，可以使用 `SINTER` 命令取得多个集合的交集：

    > sinter tag:1:news tag:2:news tag:10:news tag:27:news
    ... 运行结果 ...

除了交集，还可以使用并集、差集、随机取元素等操作。

弹出一个元素的命令叫做 `SPOP` ，用它可以很方便的对一些问题进行建模。
例如为了建立一个基于网站的扑克牌游戏，可以用集合来表示一副牌。
使用单词的第一个前缀字符表示 (C)梅花、 (D)方片、 (H)红桃、 (S)黑桃。

    >  sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
       D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
       H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
       S7 S8 S9 S10 SJ SQ SK
       (integer) 52

现在给每个玩家发五张牌。
命令 `SPOP` 随机移除一个元素并将其返回给客户端，非常适合在这里使用。

但是如果直接在牌堆里调用它的话，下一次发牌之前需要重新生成牌堆，这可能不甚理想。
所以在开始前可以先将键 `deck` 的集合拷贝至键 `game:1:deck` 。

`SUNIONSTORE` 通常用来取多个集合的并集并将结果存入另一个集合。
而通过巧妙的对单个集合自身取并集，可以获得牌堆的拷贝：

    > sunionstore game:1:deck deck
    (integer) 52

现在可以给第一个玩家发五张牌了：

    > spop game:1:deck
    "C6"
    > spop game:1:deck
    "CQ"
    > spop game:1:deck
    "D1"
    > spop game:1:deck
    "CJ"
    > spop game:1:deck
    "SJ"

对J，运气一般。。。

现在该来介绍一下取得集合中元素数量的命令了。
它在集合理论中通常叫做 *集合的势* ，所以Redis的命令叫做 `SCARD` 。

    > scard game:1:deck
    (integer) 47

进行运算：52 - 5 = 47 。

如果只想获取随机元素而不从集合中移除，可以使用命令 `SRANDMEMBER` 。
它还提供了返回重复元素与不重复元素的功能。

<a name="sorted-sets"></a>
Redis有序集合
---

有序集合就像集合与哈希的混合体。
和集合一样，有序集合的元素是唯一的、不可重复的字符串，所以从某种程度上说有序集合就是集合。

集合中的元素不是有序的，而有序集合的每个元素都与一个浮点数值进行关联，叫做 *分数* （这就是为什么它和哈希比较像，每个元素都映射了一个值）。

而且，有序集合中的元素是 *有序的* （所以它们不同于请求的顺序，排序是有序集合数据结构的特性）。
排序遵循以下规则：

* 如果元素A和B的分数不一样，那么当A.score > B.score时A > B。
* 如果A和B的分数相同，那么当A字符串的字典顺序大于B字符串时A > B。A和B的字符串不可能相等因为有序集合只包含唯一的元素。

举一个简单的例子，向有序集合中添加一些骇客的名字，以出生年作为“分数”。

    > zadd hackers 1940 "Alan Kay"
    (integer) 1
    > zadd hackers 1957 "Sophie Wilson"
    (integer) 1
    > zadd hackers 1953 "Richard Stallman"
    (integer) 1
    > zadd hackers 1949 "Anita Borg"
    (integer) 1
    > zadd hackers 1965 "Yukihiro Matsumoto"
    (integer) 1
    > zadd hackers 1914 "Hedy Lamarr"
    (integer) 1
    > zadd hackers 1916 "Claude Shannon"
    (integer) 1
    > zadd hackers 1969 "Linus Torvalds"
    (integer) 1
    > zadd hackers 1912 "Alan Turing"
    (integer) 1


可以看到 `ZADD` 类似于 `SADD` ，多接受一个分数参数（添加元素之前的那个）。
`ZADD` 的参数也是可变的，可以自由指定多个分数-值对，上面的例子里没有用到。

对于有序集合，按照出生年份的顺序返回非常简单，因为实际上它们 *已经是有序的了* 。

内部实现：有序集合通过一个双端口数据结构实现，包含一个跳表和一个哈希表，每次添加元素时Redis的操作复杂度为O(log(N))。
这还不错，并且当要求元素进行排序时Redis不需要做任何工作，它们已经是有序的了：

    > zrange hackers 0 -1
    1) "Alan Turing"
    2) "Hedy Lamarr"
    3) "Claude Shannon"
    4) "Alan Kay"
    5) "Anita Borg"
    6) "Richard Stallman"
    7) "Sophie Wilson"
    8) "Yukihiro Matsumoto"
    9) "Linus Torvalds"

注意：0和-1代表从第一个元素到最后一个元素（与命令 `LRANGE` 的-1类似）。

如果想要反方向排序，从年轻到年长，怎么办呢？
使用命令 [ZREVRANGE](/commands/zrevrange) ：

    > zrevrange hackers 0 -1
    1) "Linus Torvalds"
    2) "Yukihiro Matsumoto"
    3) "Sophie Wilson"
    4) "Richard Stallman"
    5) "Anita Borg"
    6) "Alan Kay"
    7) "Claude Shannon"
    8) "Hedy Lamarr"
    9) "Alan Turing"

分数也可以返回，使用命令 `WITHSCORES` ：

    > zrange hackers 0 -1 withscores
    1) "Alan Turing"
    2) "1912"
    3) "Hedy Lamarr"
    4) "1914"
    5) "Claude Shannon"
    6) "1916"
    7) "Alan Kay"
    8) "1940"
    9) "Anita Borg"
    10) "1949"
    11) "Richard Stallman"
    12) "1953"
    13) "Sophie Wilson"
    14) "1957"
    15) "Yukihiro Matsumoto"
    16) "1965"
    17) "Linus Torvalds"
    18) "1969"

区间操作
---

有序集合的区间操作很强大。
可以使用 `ZRANGEBYSCORE` 命令取得所有出生在1950及以前的骇客：

    > zrangebyscore hackers -inf 1950
    1) "Alan Turing"
    2) "Hedy Lamarr"
    3) "Claude Shannon"
    4) "Alan Kay"
    5) "Anita Borg"

我们让Redis返回所有分数在负无穷至1950（包含二者）的元素。

删除指定区间的元素也是可以的。
让我们从有序集合中删除出生在1940至1960年的骇客：

    > redis-cli zremrangebyscore hackers 1940 1960
    (integer) 4

`ZREMRANGEBYSCORE` 命令的名字或许不是很好记，却非常有用，而且可以返回删除元素的个数。

有序集合另一个非常有用的操作元素的命令是取得排名。
可以用它获取元素在有序集合中的排列位置。

    > zrank hackers "Anita Borg"
    (integer) 4

还有`ZREVRANK` 命令可以取得降序排列后的元素所在位置。

字典式分数
---

从近期的Redis 2.8版本开始加入了一个新特性，假定有序集合中的元素分数相同，以字典顺序取得元素区间（元素以C函数 `memcmp` 进行比较，所以确保了不会有外部调整，每个Redis实例都将得到相同的结果）。

针对字典区间的主要命令有 `ZRANGEBYLEX` 、 `ZREVRANGEBYLEX` 、 `ZREMRANGEBYLEX` 和 `ZLEXCOUNT` 。

重新来看著名骇客的例子，这次每个元素的分数皆为零：

    > zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
      "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
      0 "Linus Torvalds" 0 "Alan Turing"

由于有序集合的排序规则，他们已经是按字典排好序的了：

    > zrange hackers 0 -1
    1) "Alan Kay"
    2) "Alan Turing"
    3) "Anita Borg"
    4) "Claude Shannon"
    5) "Hedy Lamarr"
    6) "Linus Torvalds"
    7) "Richard Stallman"
    8) "Sophie Wilson"
    9) "Yukihiro Matsumoto"

使用 `ZRANGEBYLEX` 可以取得部分字典区间的元素：

    > zrangebylex hackers [B [P
    1) "Claude Shannon"
    2) "Hedy Lamarr"
    3) "Linus Torvalds"

区间可开可闭（依据第一个字符），字符 `+` 和 `-` 代表正负无穷。更多信息请阅读相关命令文档。

这个特性很重要，可以拿有序集合作为通常意义的索引使用。
例如以一个128比特的无符号整数索引元素，只需要向有序集合中插入的每个元素加上16个字节的前缀即 **128比特大端数** ，并让元素分数相同（比如0）。
大端数在字典排序（原始字节排序）时和数字排序是一样的，这样就可在128比特的范围内进行任意查询，并去掉查询结果元素的前缀。

这里有一个更实际的演示，请见 [Redis自动补全演示](http://autocomplete.redis.io) 。

更新分数：领导者布告栏
---

在进行下一话题前最后说一点。
可以随时更新有序集合的分数。
只需要对有序集合中已经存在的元素调用 `ZADD` 就能以 O(log(N)) 的时间复杂度更新它的分数（和位置）。因此，有序集合适用于写密集的场景。 

由于此特征，有一种典型的用法就是领导者布告栏。
这个经典的脸书应用程序根据用户的分数进行排序汇总，使用取得排名的操作将最前面的N个用户显示在领导者布告栏上（比如“你在分数排行榜上排名第4932位”）。

<a name="bitmaps"></a>
位图
---

位图不是一个真正的数据结构，而是在字符串类型上定义的一组面向比特的操作。
字符串是二进制安全的且最大长度为512 MB，设置2^32个比特位比较合适。

比特操作分两类：常量时间的单比特操作，比如设置比特位1或0、或取值；针对一组比特位的操作，比如计算指定区间的比特位设置数量（比如统计总数）。

位图最大的一个优点是存储信息非常节省空间。
比如有个系统的用户以自增ID进行标识，每个用户可以只用一个比特位记录某个信息（比如了解用户是否要接收新闻）总共只用512 MB内存就能记住四十亿个用户。

使用命令 `SETBIT` 和 `GETBIT` 设置和取得比特：

    > setbit key 10 1
    (integer) 1
    > getbit key 10
    (integer) 1
    > getbit key 11
    (integer) 0

命令 `SETBIT` 的第一个参数为比特位置，第二个参数为比特值，1或0。
如果比特位置超出了当前字符串的长度，命令会自动扩充字符串。

`GETBIT` 值返回指定位置的比特值。
超出范围的比特值（指向的比特位置超过了存储在目标键的字符串长度）为零。

有三个命令可操作多比特：

1. `BITOP` 对不同的字符串进行比特比较。操作包括AND 、 OR 、 XOR 和 NOT。
2. `BITCOUNT` 计算比特数，得出比特值为1的数量。
3. `BITPOS` 查找第一个比特值为0或1的比特位。

 `BITPOS` 和 `BITCOUNT` 都可以对字符串的字节区间进行操作，不用计算整个字符串。
下面是一个 `BITCOUNT` 调用的小例子：

    > setbit key 0 1
    (integer) 0
    > setbit key 100 1
    (integer) 0
    > bitcount key
    (integer) 2

位图通常用于：

* 各种实时分析。
* 高效高性能的关联对象ID与布尔信息的存储。

比如你想要了解网站日最长连续访问的用户。
以零作为网站上线的那一天开始计数，在每次用户访问网站时使用 `SETBIT` 设置比特。
使用比特值作为索引可以算出当前unix时间，减去初始偏移量，除以3600\*24。

这样每个用户用一个小字符串记录每天的访问信息。
使用 `BITCOUNT` 可以轻松取得指定用户访问网站的总天数。使用少量 `BITPOS` 或简单的在客户端取得位图并分析，可以轻松的计算最长访问。

可以非常容易的将位图拆分至多个键，比如为了数据分片，因为通常不建议使用大量的键。
拆分位图键的一个简单方法是命名键为 `比特数/M` ，每个键存储M个比特。使用 `比特数 MOD M` 在键里面查找第N个比特。

<a name="hyperloglogs"></a>
HyperLogLogs
---

HyperLogLog是一种用于记录唯一性的概率数据结构（技术上参考了估算集合势）。
通常统计唯一事物需要使用与事物总数量相称的内存，因为需要记住过去已经发现的元素以防止多次计数。
然而有一些算法通过牺牲精度来节省内存：比如在Redis中的实现，在进行估算测量后返回了错误的结果只占不到总数的1%。
算法神奇的地方是不再需要与统计事物成比例的内存，而只需要常数的内存！
HyperLogLog（下称HLL）最坏需要占用12k字节，当发现的元素很少时空间占用量非常少。

技术上来说HLL在Redis是一种特殊的数据结构，以Redis字符串进行编码。所以可以使用 `GET` 取得序列化的HLL，并使用 `SET` 反序列化到服务器。

从概念上讲HLL的API就像在使用集合做同样的事情。
使用 `SADD` 将待观察的元素放入集合，使用 `SCARD` 查询集合的独立元素数量，因为 `SADD` 不会添加重复元素。

而HLL不会真正的将元素 *添加* 进去，其数据结构只包含了是否已持有元素的状态，API则是相同的：

* 每次遇到一个新元素，使用 `PFADD` 进行计数。
* 使用 `PFCOUNT` 获得目前已使用 `PFADD` 添加进去的独立元素的势。

        > pfadd hll a b c d
        (integer) 1
        > pfcount hll
        (integer) 4

一个此数据结构的用例是统计用户每天进行不同搜索查询的数量。

Redis还可以进行HLL联合操作，更多信息请查询 [完整文档](/commands#hyperloglog) 。

其他值得注意的功能
---

Redis API还有一些重要的功能没在这篇文档中提及，不妨留意一下：

* 可以 [渐进的迭代大集合的键](/commands/scan).
* 可以 [在服务端执行Lua脚本](/commands/eval) 以降低延迟节省带宽。
* Redis还是一个[发布-订阅 服务器](/topics/pubsub)。

更多
---

此文档只是对API的一个基本介绍。更多信息请参照 [命令文档](/commands) 。

感谢阅读，祝你使用Redis愉快！

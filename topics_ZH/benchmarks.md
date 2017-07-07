# Redis有多快？

Redis包含了 `redis-benchmark` ，一个模拟N个客户端同时发送M个执行命令请求的工具（类似于Apache的 `ab` 工具）。
在下面你将会找到基准程序在Linux下的完整输出。

支持以下选项：

    用法: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

    -h <hostname>      服务器主机名（默认 127.0.0.1）
    -p <port>          服务器端口号（默认 6379）
    -s <socket>        服务器套接字（将重写主机名与端口号）
    -a <password>      Redis认证的密码
    -c <clients>       并行连接数（默认 50）
    -n <requests>      请求总数（默认 100000）
    -d <size>          SET/GET值的字节数据大小（默认 2）
    --dbnum <db>       选定指定编号的数据库（默认0）
    -k <boolean>       1=保持连接 0=重连接（默认 1）
    -r <keyspacelen>   使用随机键进行SET/GET/INCR，随机值进行SADD。使用此选项基准程序将用一个范围从0到keyspacelen-1的12位数字参数扩展字符串 __rand_int__ 。每次执行命令此值都将变化。默认的测试用它在指定范围内创建随机键。
    -P <numreq>        并行 <numreq> 请求数. 默认 1（无并行）.
    -q                 简洁的。仅显示 查询/秒 数据
    --csv              以CSV格式输出
    -l                 循环。让测试一直运行下去
    -t <tests>         仅运行以逗号分隔的列表的测试。测试的名称在产生的输出中一一对应。
    -I                 挂起模式。仅打开N个挂起的连接并进行等待。

在执行基准程序前需要运行Redis实例。
标准的例子如：

    redis-benchmark -q -n 100000

使用此工具非常简单，你也可以编写自己的基准程序，但是作为有效的基准程序，要避免一些陷阱。

只运行一部分测试
---

不必每次执行redis-benchmark都要运行所有的默认测试。
最方便的方法是使用 `-t` 选项，只选择运行一部分测试。见下面的例子：

    $ redis-benchmark -t set,lpush -n 100000 -q
    SET: 74239.05 requests per second
    LPUSH: 79239.30 requests per second

上面的例子只运行了SET和LPUSH命令，并使用简洁模式（见 `-q` 选项）。

也可以指定命令让基准程序直接执行，例如：

    $ redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
    script load redis.call('set','foo','bar'): 69881.20 requests per second

选择键空间的大小
---

默认情况下基准程序使用单个键进行测试。
由于Redis是内存内系统，人为的基准程序与真实系统的差别并不是那么大。
但仍然可以通过使用很大的键空间来加强缓存失效的影响以模拟更加真实的工作负载。

通过 `-r` 来设置。
例如，如果想用十万个随机键执行一百万次SET操作，可以使用如下命令：

    $ redis-cli flushall
    OK

    $ redis-benchmark -t set -r 100000 -n 1000000
    ====== SET ======
      1000000 requests completed in 13.86 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    99.76% `<=` 1 milliseconds
    99.98% `<=` 2 milliseconds
    100.00% `<=` 3 milliseconds
    100.00% `<=` 3 milliseconds
    72144.87 requests per second

    $ redis-cli dbsize
    (integer) 99993

使用管道
---

默认情况下客户端只有在接收到上一个命令的应答后才会发送下一个命令（基准程序在没有指定 `-c` 时模拟50个客户端），
这意味着服务器会需要一次读操作调用来读取客户端的命令。RTT也会花费时间。

Redis支持 [/topics/pipelining](管道)，可以一次发送多个命令，经常用在真实的应用程序中。
Redis管道可以显著提高服务器的吞吐量。

以下是一个在11" MacBook Air 中使用管道执行16个命令的例子：

    $ redis-benchmark -n 1000000 -t set,get -P 16 -q
    SET: 403063.28 requests per second
    GET: 508388.41 requests per second

使用管道能够显著提高性能。

陷阱与误解
---------------------------

第一点很明显：只比较同类是基准程序的黄金准则。例如可以在相同的工作负载下比较不同版本的Redis。或者比较相同版本的Redis但使用不同的选项。
如果打算把Redis与别的东西进行比较，评估功能与技术上的区别是很重要的，要把它们考虑进来。

+ Redis是一个服务端：所有的命令都牵涉网络通信或IPC往返。把它与内嵌存储应用如SQLite、Berkeley DB、Tokyo/Kyoto Cabinet等比较是没有意义的，因为大多数操作的消耗都主要来源于网络/协议的处理。
+ Redis命令对所有通用的命令都会返回一个确认。一些其他的数据存储应用没有这么做（例如MongoDB没有明确确认写操作）。比较Redis与其他储应用的单项查询，作用有限。
+ 简单循环Redis的同步命令不是在检测Redis自身，而是在检测网络（或IPC）传输。若要真实的测试Redis，需要多个连接（像redis-benchmark）并且/或者使用并行、多线程或进程，来计算若干命令的总消耗。
+ Redis是一个内存数据存储应用，并有几个持久化选项。如果打算与具有事务处理功能的服务（MySQL、PostgreSQL等）进行比较，应考虑激活AOF并选择合适的fsync策略。
+ Redis的服务是单线程的。不是设计用于多核CPU的。对于多核，如果需要请运行若干个Redis实例进行扩展。把单实例Redis与多线程数据存储应用进行对比并不是很公平。

一个普遍的误解是，redis-benchmark被设计用来让Redis的性能看起来美好，吞吐量是由redis-benchmark模拟出来的，在真实程序中则无法实现。这其实是错误的。

`redis-benchmark` 程序可以快速而有效的获取数据，估算Redis实例在指定硬件环境下的性能。
然而在默认条件下，这并不代表Redis实例所能承受的最大吞吐量。
实际上，使用管道及速度快的客户端（hiredis），可以轻松的编写出吞吐量比redis-benchmark更高的程序。
redis-benchmark的默认行为是只通过利用并发性来检验吞吐量（也就是创建多个至服务端的链接）。
如果没有通过 `-P` 参数启用，将不会使用管道或任何并行（一个连接最多同时执行一个查询，没有多线程）。
所以可以在手工执行一个后台 `BGSAVE` 操作时使用 `redis-benchmark` ，提供给用户更加 *真实* 的数据。

欲使用管道模式运行基准程序（实现更高的吞吐量），需要显式使用-P选项。
这是值得测试的，很多基于Redis的程序都积极使用管道来提高性能。
但需要注意尽量使用接近平均值的管道长度，以获得较真实的数据。

最后，基准程序应该与其他数据存储应用在相同的工作环境中比较相同的操作。把redis-benchmark与其他的基准程序做比较完全没有意义。

例如，Redis与memcached可以在单线程模式下比较GET/SET操作。
二者的数据都存储在内存中，协议层面的工作方式几乎相同。
使用相同的方式（管道）对各自的基准程序进行聚集查询，使用近似的连接数，比较的结果才有意义。

Redis (antirez) 和 memcached (dormando) 的开发者对话对此作了很好的阐述。

[antirez 1 - On Redis, Memcached, Speed, Benchmarks and The Toilet](http://antirez.com/post/redis-memcached-benchmark.html)

[dormando - Redis VS Memcached (slightly better bench)](http://dormando.livejournal.com/525147.html)

[antirez 2 - An update on the Memcached/Redis benchmark](http://antirez.com/post/update-on-memcached-redis-benchmark.html)

可以看到，最终一旦各方面技术都考虑到了的话，二者的区别并没有想象的那么大。
请注意在进行这些对比之后，Redis和memcached都进行了优化。

最终，当服务进行充分调整后（Redis或memcached肯定属于此类），让其满负荷工作可能会很难。
有时，性能瓶颈在客户端，而不是在服务端。
这种情况下，客户端（也就是基准程序自身）需要进行调整，或进行横向扩展等，以便达到最大吞吐量。

影响Redis性能的因素
-----------------------------------

有多个因素影响着Redis的性能。
之所以在这里提及，是因为它们可以改变任何基准测试的结果。
请注意即使Redis实例运行在低端、没有经过优化的机器中，通常也可以为大多数应用程序提供足够好的性能。

+ 网络带宽与时延通常直接影响性能。
在运行基准程序前，使用ping程序检测客户端与服务器主机间的时延是一个好习惯。
关于带宽，以Gbit/s为单位估算吞吐量，并与网络理论带宽进行对比通常比较有效。
例如基准程序以100000 q/s的频率向Redis设置4KB大小的字符串，实际将会消耗3.2Gb每秒的带宽，可能更适合使用10Gb每秒的网络而不是1Gb每秒。
在很多实际应用场景中，Redis的吞吐量在被CPU限制住以前就被网络限制住了。
为了能够在一个服务器上运行多个具有高吞吐量的Redis实例，值得考虑使用10Gb每秒的网卡，或结合使用多个1Gb每秒的网卡。
+ CPU是另一个非常重要的因素。由于单线程特性，Redis更偏爱有着大量缓存的快速CPU而不是多核。
目前在此项上Intel的CPU处于领先。
与Intel的Nehalem EP/Westmere EP/Sandy bridge CPU相比，Redis在相同级别的AMD Opteron CPU上罕见的只能获得一半的性能。
当客户端比如redis-benchmark与服务器端运行在同一台机器上时，CPU将成为性能的限制因素。
+ 内存的速度与带宽对全局性能的影响较小，特别是对于小对象而言。大对象（>10 KB）则可能会变得明显。
通常，为了提升Redis的性能花大价钱买昂贵而快速的内存条不是特别划算。
+ Redis在虚拟机上运行会比在相同硬件的普通服务器上要慢。 
如果可能的话推荐在物理机上运行Redis。
这并不是说Redis在虚拟机环境中很慢，相反其性能仍然很优秀。在虚拟机环境中遇到的大多数严重性能问题无非是超容量、非本地化磁盘导致的高延迟，或旧版本hypervisor缓慢的 `fork` 系统调用。
+ 当服务器端与客户端基准程序运行在同一台机器上时，TCP/IP回环与unix域套接字都可以被使用。
这依赖于所运行的平台，unix域套接字可以相对于TCP/IP回环提高50%的吞吐量（例如在Linux上）。
redis-benchmark默认使用TCP/IP回环。
+ 如果大量使用流水线（即长流水线），unix域套接字相对于TCP/IP回环的性能优势将被削弱。
+ 当在以太网中使用Redis时，使用流水线聚集调用命令将会非常高效，尤其当数据的大小在以太网数据包的大小以内时（大约1500字节）。
实际中，处理10字节、100字节或1000字节的查询，吞吐量是相同的。
请见下图。

![数据大小的影响](https://github.com/dspezia/redis-doc/raw/client_command/topics/Data_size.png)

+ 在多CPU套接字服务器中，Redis的性能依赖于NUMA的设置及进程位置。
最明显的影响是redis-benchmark的结果变得不确定，因为客户端与服务器端的进程被随机的分布在各个核中。
想要获得确定的结果，需要使用进程管理工具（在Linux上：taskset或numactl）。
最有效的组合是将客户端及服务器端始终分别放到相同CPU的两个不同的核中，以从三级缓存中获益。
下面是4KB大小的基准程序在3个服务器CPU（AMD Istanbul、Intel Nehalem EX及Intel Westmere），以不同的布置进行测试的结果。
请注意此基准程序并不是在比较CPU型号的优劣（因此没有透露CPU的具体型号及频率）。

![NUMA图表](https://github.com/dspezia/redis-doc/raw/6374a07f93e867353e5e946c1e39a573dfc83f6c/topics/NUMA_chart.gif)

+ 客户端连接数的设置也是配置是否高效的重要因素。
基于epoll/kqueue，Redis的事件管理具有非常好的伸缩性。
Redis在有多于60000个连接的基准测试中，仍然能维持50000 q/s的吞吐量。
以经验来说，一个有着30000连接数的实例只能达到有着100连接数的实例的一半吞吐量。
下面的例子展示了Redis实例的连接数与吞吐量的关系：

![connections chart](https://github.com/dspezia/redis-doc/raw/system_info/topics/Connections_chart.png)

+ 可以通过调整NIC(s)和相关中断进行高级配置来达到更高的吞吐量。
最佳的吞吐量是通过调节好Rx/Tx NIC队列与CPU核的关系，并开启RPS（Receive Packet Steering）支持来达到的。
更多信息请见
[这里](https://groups.google.com/forum/#!msg/redis-db/gUhc19gnYgc/BruTPCOroiMJ).
在有大对象的场合中使用巨型帧技术也可以使性能得到提升。
+ 根据平台的不同，Redis可以被编译出不一样的内存分配器（libc malloc，jemalloc或tcmalloc），可能在速度、内外部碎片上的行为表现中存在差异。
如果不是手动编译的Redis，可以使用INFO命令查看 `mem_allocator` 字段。
请注意大多数基准测试的运行时间都不够长，不足以产生显着的外部碎片（与生产环境的Redis实例相反）。

其他需要考虑的事
------------------------

获得稳定的测试结果是任何基准程序的重要目标，所以可以对测试结果进行对比。

+ 尽可能的尝试在独立的硬件上运行测试是一个好习惯。
如果办不到，就必须对系统进行监控以确保基准程序不会受到外部活动的影响。
+ 有些配置（普通PC机和笔记本确定有，一些服务器上也有）提供了可改变CPU核频率的变量，为操作系统级别的配置。
一些CPU型号在调整核频率以后将变得更为强劲。
为了得到可以重现的结果，最好将所有CPU核的频率尽可能调高以进行基准测试。
+ 估算运行基准程序时的系统开销很重要。
系统必须要有足够的内存，不能发生swap。
在Linux上，不要忘了设置正确的 `overcommit_memory` 参数。
请注意32位和64位的Redis实例内存占用是不同的。
+ 如果打算在基准测试中使用RDB或AOF，请确认系统中没有其他I/O操作。
避免将RDB或AOF文件存入NAS或NFS文件系统，以及其他影响网络带宽及/或延迟的设备（例如Amazon EC2的EBS）。
+ 设置Redis的日志级别（参数loglevel）为警告或注意。避免将生成的日志文件放在远程文件系统上。
+ 避免使用有可能改变基准测试结果的监控工具。
例如使用INFO定期收集统计数据很可能没有问题，但使用MONITOR将会显著影响性能测试的精确性。

# 在各种虚拟机服务器上的测试结果。

注意：以下测试已经是几年前在旧硬件上执行的了，这部分需要更新。你可以认为如今的执行结果能够达到两倍效果。
并且，Redis 4.0的性能已经在很多地方超过了2.6。

* 50个并发客户端执行两百万个请求。
* 使用Redis 2.6.14 。
* 使用loopback接口。
* 使用一百万个键。
* 16个命令使用管道，其余命令不使用管道。

**Intel(R) Xeon(R) CPU E5520  @ 2.27GHz (使用流水线)**

    $ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -P 16 -q
    SET: 552028.75 requests per second
    GET: 707463.75 requests per second
    LPUSH: 767459.75 requests per second
    LPOP: 770119.38 requests per second

**Intel(R) Xeon(R) CPU E5520  @ 2.27GHz (不使用流水线)**

    $ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q
    SET: 122556.53 requests per second
    GET: 123601.76 requests per second
    LPUSH: 136752.14 requests per second
    LPOP: 132424.03 requests per second

**Linode 2048 实例 (使用流水线)**

    $ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q -P 16
    SET: 195503.42 requests per second
    GET: 250187.64 requests per second
    LPUSH: 230547.55 requests per second
    LPOP: 250815.16 requests per second

**Linode 2048 实例 (不使用流水线)**

    $ ./redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q
    SET: 35001.75 requests per second
    GET: 37481.26 requests per second
    LPUSH: 36968.58 requests per second
    LPOP: 35186.49 requests per second

## 更多详细的不使用流水线的测试

    $ redis-benchmark -n 100000

    ====== SET ======
      100007 requests completed in 0.88 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    58.50% <= 0 milliseconds
    99.17% <= 1 milliseconds
    99.58% <= 2 milliseconds
    99.85% <= 3 milliseconds
    99.90% <= 6 milliseconds
    100.00% <= 9 milliseconds
    114293.71 requests per second

    ====== GET ======
      100000 requests completed in 1.23 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    43.12% <= 0 milliseconds
    96.82% <= 1 milliseconds
    98.62% <= 2 milliseconds
    100.00% <= 3 milliseconds
    81234.77 requests per second

    ====== INCR ======
      100018 requests completed in 1.46 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    32.32% <= 0 milliseconds
    96.67% <= 1 milliseconds
    99.14% <= 2 milliseconds
    99.83% <= 3 milliseconds
    99.88% <= 4 milliseconds
    99.89% <= 5 milliseconds
    99.96% <= 9 milliseconds
    100.00% <= 18 milliseconds
    68458.59 requests per second

    ====== LPUSH ======
      100004 requests completed in 1.14 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    62.27% <= 0 milliseconds
    99.74% <= 1 milliseconds
    99.85% <= 2 milliseconds
    99.86% <= 3 milliseconds
    99.89% <= 5 milliseconds
    99.93% <= 7 milliseconds
    99.96% <= 9 milliseconds
    100.00% <= 22 milliseconds
    100.00% <= 208 milliseconds
    88109.25 requests per second

    ====== LPOP ======
      100001 requests completed in 1.39 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    54.83% <= 0 milliseconds
    97.34% <= 1 milliseconds
    99.95% <= 2 milliseconds
    99.96% <= 3 milliseconds
    99.96% <= 4 milliseconds
    100.00% <= 9 milliseconds
    100.00% <= 208 milliseconds
    71994.96 requests per second

注意：将payload从256改变至1024或4096并不能显著改变测试结果（而因此应答包的单位升至1024字节，设置大的payload时GET将可能变得更慢）。
从50个客户端至256个客户端，我获得的测试结果相同。
只有10个客户端的时候结果开始变得稍慢。

可以预料到不同的机器能够测试出不同的结果。
例如低性能的机器像 *Intel core duo T5500 1.66 GHz，运行Linux 2.6* 将输出如下结果：

    $ ./redis-benchmark -q -n 100000
    SET: 53684.38 requests per second
    GET: 45497.73 requests per second
    INCR: 39370.47 requests per second
    LPUSH: 34803.41 requests per second
    LPOP: 37367.20 requests per second

另一个使用64位系统、Xeon L5420 2.5 GHz的机器：

    $ ./redis-benchmark -q -n 100000
    PING: 111731.84 requests per second
    SET: 108114.59 requests per second
    GET: 98717.67 requests per second
    INCR: 95241.91 requests per second
    LPUSH: 104712.05 requests per second
    LPOP: 93722.59 requests per second

# 其他Redis基准测试工具

这里有一些第三方的Redis基准测试工具，关于工具的目标及功能等具体信息请查阅其文档。

* [Redis Labs](https://twitter.com/RedisLabs) 的 [memtier_benchmark](https://github.com/redislabs/memtier_benchmark) 是一个NoSQL Redis和Memcache的生成访问及基准测试工具。
* [Twitter](https://twitter.com/twitter) 的 [rpc-perf](https://github.com/twitter/rpc-perf) 是一个RPC服务基准测试工具，支持Redis和Memcache。
* [Yahoo @Yahoo](https://twitter.com/Yahoo) 的 [YCSB](https://github.com/brianfrankcooper/YCSB) 是一个基准测试框架，支持很多数据库包括Redis。

# 在经过了优化的高端服务器硬件下的基准测试结果例子

* Redis 版本 **2.4.2**
* 默认连接数, payload 大小 = 256
* Linux机器运行于 *SLES10 SP3 2.6.16.60-0.54.5-smp*, CPU为2颗 *Intel X5670 @ 2.93 GHz*。
* 用一个核运行Redis服务器端，另一个核运行基准测试客户端并执行命令文本。

使用unix域套接字：

    $ numactl -C 6 ./redis-benchmark -q -n 100000 -s /tmp/redis.sock -d 256
    PING (inline): 200803.22 requests per second
    PING: 200803.22 requests per second
    MSET (10 keys): 78064.01 requests per second
    SET: 198412.69 requests per second
    GET: 198019.80 requests per second
    INCR: 200400.80 requests per second
    LPUSH: 200000.00 requests per second
    LPOP: 198019.80 requests per second
    SADD: 203665.98 requests per second
    SPOP: 200803.22 requests per second
    LPUSH (again, in order to bench LRANGE): 200000.00 requests per second
    LRANGE (first 100 elements): 42123.00 requests per second
    LRANGE (first 300 elements): 15015.02 requests per second
    LRANGE (first 450 elements): 10159.50 requests per second
    LRANGE (first 600 elements): 7548.31 requests per second

使用TCP/IP回环：

    $ numactl -C 6 ./redis-benchmark -q -n 100000 -d 256
    PING (inline): 145137.88 requests per second
    PING: 144717.80 requests per second
    MSET (10 keys): 65487.89 requests per second
    SET: 142653.36 requests per second
    GET: 142450.14 requests per second
    INCR: 143061.52 requests per second
    LPUSH: 144092.22 requests per second
    LPOP: 142247.52 requests per second
    SADD: 144717.80 requests per second
    SPOP: 143678.17 requests per second
    LPUSH (again, in order to bench LRANGE): 143061.52 requests per second
    LRANGE (first 100 elements): 29577.05 requests per second
    LRANGE (first 300 elements): 10431.88 requests per second
    LRANGE (first 450 elements): 7010.66 requests per second
    LRANGE (first 600 elements): 5296.61 requests per second

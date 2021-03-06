# FAQ

## 为什么Redis和其他键-值存储是不一样的？

主要有两个原因。

* Redis是键-值数据库的一种进化，值可以是复杂的数据类型，并以原子方式进行操作。Redis的数据类型非常接近于编程的基本数据类型，不需要额外的抽象层。
* Redis是一个注重内存而非持久化磁盘的数据库，它代表了一种不同的风格，要想获得非常高的读写速度，数据集的大小不能超过内存。
内存数据库的另一个优势是在内存中操纵复杂的数据结构要比在磁盘中简单得多，所以Redis可以轻松的实现很多复杂的数据结构。
同时两种磁盘保存格式（RDB和AOF）就无需适用于随机访问，以只可追加的方式保存即可（甚至新版本的AOF重写也是只可追加操作，是从内存中拷贝数据并生成的）。
然而这种设计也引入了不同于传统磁盘存储的挑战。主要数据放于内存，Redis的操作必须小心谨慎，确保所有更新都会同步到磁盘中。

## Redis的内存占用情况如何？

这里有一些例子（使用64位实例）：

* 空的实例使用约3MB内存。
* 1百万对儿短的键 -> 字符串值，使用约85MB内存。
* 1百万对儿键 -> 哈希对象，包含5个字段，使用约160MB内存。

可以使用 `redis-benchmark` 工具进行测试，生成随机数据集后使用命令 `INFO memory` 查看空间占用情况。

64位系统存储相同的键会使用比32位系统多的多的内存，尤其在键和值比较小的情况下。因为指针在64位系统中会占用8个字节。
诚然，64位系统支持大量的内存。所以为了运行大型Redis服务器，或多或少会需要64位系统。
另外一种选择是分片。

## 我喜欢Redis的高级操作及特性，但不喜欢它把所有东西都放在内存里而数据集不能超过内存大小。能不能改一下呢？

过去Redis的开发人员曾经尝试使用虚拟内存或其他系统以允许数据集超过内存大小，但是最终我们很高兴的达成了共识：内存用于提供数据服务，磁盘用于提供数据存储。
所以现在没有为Redis创建后端磁盘服务的计划。Redis仍将保持它的设计理念。

如果总的内存量不是问题，而是要将数据集拆分至多个Redis实例的话，请阅读 [分区](/topics/partitioning) 文档。

最近Redis的开发赞助商Redis Labs开发了一个称为 "Redis on flash" 的功能，可以混合使用RAM/flash设置访问偏好模式，应对更大的数据集。
你可以查阅他们的资料，这并不包括在Redis开源代码中。

## 结合存储于磁盘的数据库使用Redis怎么样？

挺好的，将写密集的较小数据（和那些能够有效使用Redis数据结构解决问题的数据）放在Redis里，将较大数据如 *blobs* 使用SQL存到磁盘数据库上。
类似于有时Redis用作磁盘数据库数据的内存镜像。
这更像缓存应用，但也更高级，通常Redis的数据集会随着磁盘DB数据集一起更新。并且不会导致缓存未命中的刷新。

## 可不可以减少Redis的内存使用？

如果可以的话使用32位的Redis实例。并充分利用小的哈希、列表、有序集合和整型值集合，在元素数量较少的情况下Redis可以以一种更紧凑的方式表示这些数据类型。
更多信息请参照 [内存优化](/topics/memory-optimization) 。

## Redis运行时发生内存不足会怎样？

Redis进程可能会被Linux内核管理杀掉、出错并崩溃，或开始变得缓慢。
在当今流行的操作系统中malloc()并不一定返回NULL。通常当内存不足时服务器会开始进行磁盘和内存的数据交换，Redis的性能也会下降，这时基本上你就能察觉到出问题了。

Redis有保护机制允许用户设置最大内存使用量，通过在配置文件中使用选项 `maxmemory` ，来限制Redis的内存使用量。
当到达限制时Redis将会开始对写操作命令返回错误（但会继续接收只读命令）。
在把Redis用作缓存服务器时，也可以配置当到达最大内存限制时使用键驱逐策略。

这有一篇指导如何 [把Redis作为LRU缓存](/topics/lru-cache) 的文章。

INFO命令会报告Redis使用的内存总数，所以你可以写个脚本来监控Redis服务器，检查紧急状况。

## 在Linux下后台保存失败报fork()错误，即使我还有很多可用内存！

简短的回答： `echo 1 > /proc/sys/vm/overcommit_memory` :)

详细原因：

在现代操作系统中，Redis在后台保存数据依赖于分支写拷贝：Redis分支（创建一个子进程）是父进程的一个完整拷贝。
子进程将DB保存至磁盘并最终退出。
作为父进程的拷贝，理论上子进程也会消耗相同大小的内存。但是实际长受益于大多数现代操作系统的写拷贝实现，父子进程会 _共享_ 公共内存页。
只有页在父或子进程中被修改了，它才会进行复制。
由于在子进程进行保存时理论上所有的页都可能会被修改，Linux无法预先知道子进程将会消耗多少内存。
所以如果将 `overcommit_memory` 设为零分支进程将会失败，除非有足够多的内存复制所有父进程的内存页。
即如果你的Redis数据集有3GB大小而只有2GB可用内存，执行将会失败。

将 `overcommit_memory` 设为1降低对Linux的要求，使用一种更乐观的分配方式执行分支保存，对Redis无疑是个好选择。

Red Hat杂志的 ["理解虚拟内存"][redhatvm] 是一个很好的了解Linux虚拟内存工作原理、及 `overcommit_memory` 和 `overcommit_ratio` 的经典文章。
注意，文章中对 `overcommit_memory` 的 `1` 和 `2` 的配置值弄反了：可用值的正确含义请参照 [proc(5)][proc5] 帮助页。

[redhatvm]: http://www.redhat.com/magazine/001nov04/features/vm/
[proc5]: http://man7.org/linux/man-pages/man5/proc.5.html

## Redis的磁盘镜像操作是原子的么？

是的，redis后台的保存操作总是是在服务器执行命令之外创建分支进行的，所以每个在内存中的原子命令在磁盘快照的观察点中也是原子的。

## Redis是单线程的，如何才能利用多CPU / 核？

CPU不太可能成为Redis的瓶颈，因为通常Redis的瓶颈是内存或者网络。
例如使用流水线Redis运行在一个普通的Linux系统上可以达到一百万个请求每秒，所以如果应用程序主要使用 O(N) 或 O(log(N)) 的命令的话，将几乎不会使用多少CPU。

如果想最大化CPU的使用率，可以在同一台机器中启动多个Redis实例，并当成不同的服务器。
一些观点认为单机终归是不行的，所以如果你想要使用多CPU，可以早点开始考虑进行数据分片。

更多关于使用多个Redis实例的信息请参见文章 [分区](/topics/partitioning) 。

从4.0开始Redis将会逐渐开发多线程功能。
目前仅限于后台删除对象，该命令可以阻塞住用Redis module开发的命令。
下一个版本将计划引入更多多线程的功能。

## 一个Redis实例所能保存的最大键数是多少？哈希表、列表、集合、有序集合所能保存的的最大元素数又是多少？

Redis可以保存最多2^32个键，在实际测试中单个实例至少可以保存2.5亿个键。

每个哈希表、列表、集合、有序集合可以保存2^32个元素。

换句话说，这个限制相当于系统的可用内存。

## 从服务与主服务的键数不一样，为什么？

这在设置了键的生存时间（Redis键超时机制）时是正常现象，因为：

* 主服务在第一次与从服务同步时会生成一个RDB文件。
* 这个RDB文件不会包含在主服务中已经超时的键，但这些键仍然在内存里。
* 虽然逻辑上已经过期了，这些超时的键仍然存在于Redis主服务中，内存会逐渐进行增量回收或在显式访问时主动回收。
* 当从服务读取主服务生成的RDB文件时，这些超时的键是不会被加载进去的。

所以，在使用键超时的时候从服务的键少于主服务是很正常的事，主从实例不会有逻辑上不一致的内容。

## Redis的字面意思是什么？

远程词典服务器。

## 为什么要开发Redis这个项目？

最初Redis是为了扩展 [LLOOGG][lloogg] 而创建的。
在实现了服务的基本功能以后，我想把它拿出来与别人进行分享，所以Redis就变成了开源项目。

[lloogg]: http://lloogg.com

## Redis如何发音？

颜色的"red"加"iss"。

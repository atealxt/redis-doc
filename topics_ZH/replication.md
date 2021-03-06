复制
===

Redis复制是一个非常容易配置和使用的主从复制功能，实现了Redis主服务至从服务的精确拷贝。
下面是一些关于Redis复制的重要说明：

* Redis使用异步复制。从Redis 2.8开始，从服务会周期性的报告复制流中已处理的数据数量。

* 主服务可以有多个从服务。

* 从服务可以接受其他从服务连接。除了一个主服务可以连接多个从服务之外，从服务之间也可以用类似级联的结构进行连接。

* Redis复制在主服务端是非阻塞的。在一个或多个从服务执行初始化同步时，主服务可以继续响应请求。

* Redis复制在从服务端也是非阻塞的。从服务在执行初始化同步时可以使用旧版本的数据集继续响应请求，这是默认设置。
或者可以配置Redis从服务在与主服务的连接中断时向客户端的发送一个错误。
但是在初始化同步后，需要删除旧的数据集并装载新的。这一刻从服务会阻塞住外来连接（对于很大的数据集可能长至数十秒）。

* 复制也可以被用来做水平扩展，多个从服务提供只读查询服务（例如，缓慢的 O(N) 操作可以放到从服务上进行），或简单的把从服务作为数据冗余。

* 可以使用复制避免主服务端的数据持久化开销：经典的做法是配置主服务 `redis.conf` ，避免持久化至磁盘，而在从服务上进行数据保存或开启AOF。
使用这种配置时要小心。主服务重启后将会从空数据集重启开始，如果这时从服务同步它的的话，数据集也将会清空。

关闭主服务持久化时的复制安全性
---

在使用Redis复制时，强烈推荐开启主服务的持久化。如果无法这么做的话，比如考虑到延迟，应该配置实例在机器重启后 **不要自动重启** 。

为了更好的理解主服务在关闭复制时自动重启这一危险行为，在下面这个失败的场景中，主从的所有数据都丢失了：

1. 节点A作为主服务并关闭持久化，节点B和C从A复制数据。
2. A节点的服务发生崩溃，通过自动重启系统重启了进程。由于其关闭了持久化，节点的数据都丢失了。
3. 而后节点B和C从A复制数据，将自己的数据集清空。

当配置高可用性的Redis Sentinel，关闭主服务持久化并开启自动重启功能仍然是危险的。
例如主服务的重启足够快Sentinel没能发现错误，上面的失败场景仍然会发生。

当在主服务关闭持久化的场景下使用复制时，如果数据的安全性很重要，应禁用实例的自动重启。

Redis复制是如何工作的
---

如果设置了主从，从服务将会通过连接向主服务发送一个SYNC命令进行同步。

如果这是一个重连接并且主服务有足够的 *日志记录* ，主服务只会发送与从服务数据不同的部分（从服务没有的）。
否则将会触发一次 *完整重同步* 。 

当触发完整重同步时，主服务启动一个后台保存进程用于生成RDB文件，同时开始缓存从客户端收到的所有写命令。 
后台文件生成后，主服务将其传送至从服务。
从服务先将其保存在磁盘上，然后装入内存。
然后主服务将所有缓存命令发送至从服务。
此操作使用Redis协议中的流式命令完成。

可以通过telnet来试验一下。
在Redis主服务正在工作时连接到从服务并发出 `SYNC` 命令进行观察。
将会看到主服务同步过来的一系列命令在telnet会话中依次重新执行。

当主从连接由于某些原因失效时，从服务可以自动重连。
即使主服务接收到了多个从服务并发的同步请求，也只会执行一次后台保存。

无磁盘依赖复制
---

通常一次完整的重同步需要先创建RDB文件至磁盘，再将它载入从服务的数据集。

对于磁盘性能不高的主服务来说这个操作可能会有很大的压力。
Redis 2.8.18是第一个支持无磁盘依赖复制功能的版本，在此配置下子进程直接将RDB发送至从服务，不会使用磁盘临时保存文件。

配置
---

配置复制非常简单：仅需要在从服务配置文件中加一行：

    slaveof 192.168.1.1 6379

当然你需要将192.168.1.1 6379替换为你的主服务的IP地址（或主机名）和端口。
或者可以调用 `SLAVEOF` 命令让主服务开始与从服务同步。

还有一些参数可以调整主服务复制时执行部分重同步的内存使用。
更多信息请见Redis发行版中的 `redis.conf` 例子文件。

可以使用配置参数 `repl-diskless-sync` 启用无磁盘依赖复制。
参数 `repl-diskless-sync-delay` 用来控制等待更多从服务加入的延迟传送时间。
更多信息请查阅Redis发行版中的 `redis.conf` 例子文件。

只读从服务
---

自Redis 2.6开始从服务支持只读模式且默认是开启状态。
此行为由redis.conf文件的 `slave-read-only` 选项控制，并且可以在运行时使用 `CONFIG SET` 开启或禁用。

只读从服务会拒绝所有写操作，这样就不会误将数据写入从服务。
这不代表此功能设计用于将从实例暴露给有不信任客户端存在的因特网或更广泛的网络，因为 `DEBUG` 或 `CONFIG` 等管理员级别的命令仍然是可用的。
但是可以使用 `rename-command` 直接在redis.conf中配置某些命令禁止使用，提高只读实例的安全性。

也许你想知道为什么可以改变只读设置让从实例可写。
这是因为虽然写入的数据会在重同步或从服务重启后被覆盖，仍然可能有一些临时数据需要存放在从服务上。

例如执行缓慢的set或zset操作并保存在本地键，就是一个可写从服务会多次遇到的情况。

要注意 **4.0之前的版本，可写从服务虽然可以对键设置过期，但是无法设定生存时间**。 
这意味着，如果使用了 `EXPIRE` 或其他命令对键设置了TTL，该键会造成内存泄漏。虽然使用读取命令不会再看到它，但仍然算在键的计数中，一直占用着内存。
所以一般在混合着只读与可写的从服务集群（4.0版本以前）中，使用有TTL的键是有问题的。

Redis 4.0 RC3或更新版本完全解决了这个问题，可写从服务可以像主服务一样使用TTL淘汰键了，只要键所在的DB序号不大于63（Redis的默认实例只有16个）。

另外值得注意的是，从Redis 4.0开始从服务的写只在本地，不会再传播至下级从服务了。
下级从服务会接收到与当前从服务自顶层主服务接收到的复制流一样的数据。
例如下面的配置：

    A ---> B ---> C

即使 `B` 是可写的，C也看不到 `B` 的写操作，而是获得和主服务 `A` 一样的数据集。

从服务连接主服务的权限设置
---	

如果主服务通过 `requirepass` 设置了密码，可以很容易的配置从服务在所有同步操作中使用该密码。

在运行的实例中，使用 `redis-cli` 并输入：

    config set masterauth <password>

如欲持久化设置，将此行加到配置文件中：

    masterauth <password>

只在有N个以上可用的复制结点时接受写操作
---

从Redis 2.8开始，可以配置Redis主服务只在有N个以上有效的从服务连接时接受写请求。

然而因为Redis使用的是异步复制，无法确保从服务接收到写操作，所以有可能发生数据丢失。

写操作是这样工作的：

* Redis从服务每秒ping一次主服务，通知复制流执行的情况。
* Redis主服务将会记住每个从服务最后一次ping的时间。
* 用户可以配置从服务的最大延迟秒数。

如果至少有N个从服务的延迟小于M秒，则认为写操作成功。

在CAP理论中可以认为这是一个的宽松版的 "C" ，虽然写操作不能完全保证一致性，但至少数据丢失的时间窗口被限制在了指定秒数内。

如果条件未满足，主服务将会返回一个错误，写操作将不被接受。

共有两个配置参数可在此特性中使用：

* min-slaves-to-write `<number of slaves>`
* min-slaves-max-lag `<number of seconds>`

更多信息请查阅 `redis.conf` 例子文件，此文件在Redis源码项目里。

Redis复制如何处理过期键
---

Redis的过期机制可以限制键的生存时间。
这样的功能依赖于实例计时的能力，而Redis从服务能够正确复制有超时设置的键，甚至用Lua脚本修改过的键。

为了实现此功能，Redis不能依赖于主从服务的时钟同步，这会导致抢占及数据集分散等无法修复的问题，所以Redis主要使用了三种技术实现了超时键的复制：

1. 从服务不做键过期处理，而是交由主服务进行。当主服务的键过期了（或经由LRU淘汰），主服务将向所有从服务发送一个 `DEL` 命令。
2. 然而因为由主服务驱动的超时机制，有时候主服务没能及时发送 `DEL` 命令，从服务内存仍会存在逻辑上已过期的键。
为了解决此问题，从服务 **只针对读操作** 使用自己的逻辑时钟对这种不应存在的键进行报告，而不会破坏数据集的一致性（从主服务来的命令终会到来）。 
这样一来从服务就不会报告逻辑超时的键仍旧存在了。
具体来说，从服务使用了一个HTML片段缓存来记录那些已经过期了却仍然存在的元素。
3. 在执行Lua脚本时不对键做超时处理。
当Lua脚本运行时，概念上主服务的时间被冻结了，所以指定键在整个脚本执行过程时间内要么存在，要么就不存在。 
这样就避免了键在脚本执行过程中超时，脚本在发送到从服务后也能对数据集起到相同的效果。

一旦从服务变为了主服务，它将独自开始键淘汰，不需要旧主服务的帮忙。

复制在Docker和NAT环境中的配置
---

当在Docker及其他使用端口转发的容器、或网络地址转换（NAT）中使用Redis复制时需要格外小心，特别在使用Redis Sentinel或类似系统中以主服务 `INFO` 或 `ROLE` 命令扫描发现从服务地址的场合。

因为在主服务执行的命令 `ROLE` ，和 `INFO` 的复制部分的输出，会显示从服务连接主服务时的IP地址，在NAT环境下可能会和从服务的逻辑地址（客户端用于连接从服务的地址）不同。

同样的，`redis.conf` 配置中的从服务监听端口，可能和转发后的重映射端口不同。

为了修复这两个缺陷，从Redis3.2.2开始可以强制指定从服务使用指定IP和端口访问主服务。
该两项的配置方法为：

    slave-announce-ip 5.5.5.5
    slave-announce-port 1234

近期发布的Redis中的例子 `redis.conf` 里也可以找到相关文档。

INFO命令和ROLE命令
---

Redis有两个命令提供了主从实例当前复制参数的详细信息。 `INFO`是其中一个。
如果调用命令为 `INFO replication` ，则显示与复制相关的信息。
另一个命令 `ROLE` 则更偏低层一些，显示主从复制状态，包括复制偏移量、连接的从服务实例等信息。

重启或故障恢复时的部分重同步
---

Redis从4.0版本开始，在故障恢复后实例提升为主服务时，仍然可以和旧的主服务进行部分重同步。
为此，从服务会记住旧主服务的复制ID和偏移量，以此提供给从服务们对旧复制的部分复制信息。

然而由于组成了新的数据集，从服务提升后的新复制ID将会是一个不同的值。
例如，旧主服务有可能可以恢复可用性，重新接受写操作，新主服务再用相同的复制ID将会违反一对复制ID与偏移量只对应一个数据集的规则。

在从服务正常关机重启后，是可以将与主服务进行重同步的信息存储至 `RDB` 文件中的。
这在系统升级时会用到。
此时，最好使用 `SHUTDOWN` 命令以在从服务上成功执行一次 `save & quit` 操作。

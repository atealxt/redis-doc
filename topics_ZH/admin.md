Redis管理
===

本页面的主题为Redis实例管理。
话题以FAQ的形式展现。新的话题将在今后陆续添加。

Redis安装提示
-----------------

+ 建议使用 **Linux操作系统** 进行部署。在OS X下我们也对Redis进行了非常多的测试，并在FreeBSD和OpenBSD下进行了一定程度的测试。然而所有主要的压力测试都是在Linux下进行的，确保了该系统下绝大多数生产部署的正常工作。
+ 请确认Linux内核的 **overcommit内存设置为1** 。添加 `vm.overcommit_memory = 1` 至 `/etc/sysctl.conf` 之后重启，或执行立即生效的命令 `sysctl vm.overcommit_memory=1` 。
* 请确认禁用Linux内核的 *巨大页透明化* 功能，它会严重影响内存的使用并带来延迟。通过这个命令来禁用： `echo never > /sys/kernel/mm/transparent_hugepage/enabled` 。
+ 请确认为系统 **提供一些交换空间**（建议大小和内存一样）。如果Linux没有交换空间并且Redis实例瞬间消耗了太多内存，Redis将因内存溢出而崩溃，或者Linux内核的OOM killer将杀掉Redis进程。
+ Set an explicit `maxmemory` option limit in your instance in order to make sure that the instance will report errors instead of failing when the system memory limit is near to be reached.
+ 如果把Redis用于写操作密集应用程序中，当保存RDB文件至磁盘或重写AOF日志时 **Redis可能会使用最多2倍于普通的内存空间消耗** 。内存的额外消耗量与在保存进程中修改的内存页数量成正比，通常与此过程中访问过的键的数量（或类型项目总数）成正比。请确认并分配相应的内存。
+ 当运行在虚拟光驱类软件中请使用 `daemonize no` 。
+ 即使禁用了持久化，在复制时Redis仍然需要保存RDB，除非使用新的无磁盘依赖复制特性，它现在处于实验阶段。
+ 如果开启了复制，请确保开启主服务的持久化，否则在其崩溃后不会自动重启：从服务会尝试成为主服务，所以如果主服务以空数据集进行重启的话，从服务的数据也会被清空。
+ Redis默认不需要 **任何鉴权设定和监听所有网络接口** 。
如果直接暴露在互联网或可能的攻击者面前将是个很大的安全问题，危险程度见这个 [攻击例子](http://antirez.com/news/96) 。
关于如何保障Redis的安全，请阅读 [安全](/topics/security) 和  [快速入门](/topic/quickstart) 文档。

在EC2上运行Redis
--------------------

+ 使用基于HVM的实例，不要用基于PV的实例。
+ 不要使用旧系列的实例，比如：应该用HVM的m3.medium来替代PV的m1.medium。
+ 在 **EC2 EBS卷** 上使用Redis持久化时要格外注意，有些时候EBS卷会有很高的延迟。
+ 如果主从服务同步出现问题，可以试试新的 **无磁盘依赖复制** （现处于实验阶段）。 

在不停止服务的前提下升级或重启Redis实例
-------------------------------------------------------

Redis被设计为可在服务器长时间运行的进程。
例如可以使用[CONFIG SET command](/commands/config-set)修改许多配置选项，而无需任何类型的重启。

从Redis 2.2开始甚至可以在不重启的情况下将持久化方式从AOF切换至RDB快照或其他方式。更多信息请查看命令 `CONFIG GET *` 的输出。

然而有些时候还是需要重启的，比如升级Redis的程序至新版本，或修改到目前为止CONFIG命令还不支持的配置参数。

以下步骤提供了一种非常通用的方法来避免停止服务。

* 将新的Redis实例设置为当前Redis实例的从服务。为此需要一个新的服务器，或足够支持同时运行两个实例的内存。
* 如果使用单个服务器，请确认从服务器运行在与主实例不同的端口上，否则从服务将无法运行。
* 等待完成复制初始化同步（检查从服务日志文件）。
* 使用INFO确认主服务与从服务的键数相同。使用redis-cli调用命令检查从服务是否工作正常。
* 可以使用 **CONFIG SET slave-read-only no** 允许从服务写入数据
* 配置所有客户端使用新的实例（也就是从服务）。
* 一旦确信主服务不再收到任何查询（可以使用[MONITOR command](/commands/monitor)进行验证），使用命令 **SLAVEOF NO ONE** 将从服务切成主服务，并停止原主服务。

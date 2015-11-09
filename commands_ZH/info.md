 `INFO` 命令以一种易于计算机解析及人理解的格式返回服务器的信息和统计数据。

可以使用下列可选参数来选取指定部分的信息：

*   `server`: 生成关于Redis服务器的信息
*   `clients`: 客户端连接的信息
*   `memory`: 内存消耗的相关信息
*   `persistence`: RDB和AOF的相关信息
*   `stats`: 一般性统计
*   `replication`: 主/从复制信息
*   `cpu`: CPU消耗的统计信息
*   `commandstats`: Redis命令的统计信息
*   `cluster`: Redis集群的信息
*   `keyspace`: 数据库相关的统计信息

也可以使用以下参数：

*   `all`: 返回所有部分的信息
*   `default`: 只返回默认部分的信息

当没有提供参数时，采用 `default` 选项。

@return

@bulk-string-reply: 一个文本行集合。

行可以包含信息种类的名称（以#字符开头）或一条信息。
所有信息的格式为 `字段名:值` 并以 `\r\n` 结束。

```cli
INFO
```

## 注意

请注意根据版本的不同Redis添加或删除了某些属性。强健的客户端程序应该在解析命令的时候跳过未知的属性，并优雅的处理丢失的属性。

以下是Redis >= 2.4 版本以后的字段描述。


以下是所有 **server** 字段的含义：

*   `redis_version`: 服务器版本
*   `redis_git_sha1`:  Git SHA1
*   `redis_git_dirty`: Git脏标志位
*   `os`: 运行Redis服务的操作系统
*   `arch_bits`: 架构 (32 or 64 bits)
*   `multiplexing_api`: Redis使用的事件循环程序
*   `gcc_version`: 编译Redis服务使用的GCC编译器版本
*   `process_id`: 服务器进程PID
*   `run_id`: 用来确认Redis服务器身份的随机值（供守护服务器和集群簇服务器使用）
*   `tcp_port`: TCP/IP监听端口
*   `uptime_in_seconds`: Redis服务启动后经过的秒数
*   `uptime_in_days`: 用天来表示该值
*   `lru_clock`: 每分钟进行自增的时钟，供LRU管理器使用

以下是所有 **clients** 字段的含义：

*   `connected_clients`: 客户端连接数（从服务器连接除外）
*   `client_longest_output_list`: 当前客户端连接中的最长输出列表
*   `client_biggest_input_buf`: 当前客户端连接中的最大输入缓冲
*   `blocked_clients`: 处于阻塞调用中的客户端数（BLPOP, BRPOP, BRPOPLPUSH）

以下是所有 **memory** 字段的含义：

*   `used_memory`:  Redis已占用的内存分配字节数（标准的 **libc**, **jemalloc** ，或替代的分配器如 [**tcmalloc**][hcgcpgp]）
*   `used_memory_human`: 易读的内存占用数
*   `used_memory_rss`: 通过操作系统报告的Redis已占用的内存分配字节数（即进程内存占用数）。数据由诸如 `top(1)` 和 `ps(1)` 的工具中得到。
*   `used_memory_peak`: Redis内存消耗峰值（字节数）
*   `used_memory_peak_human`: 易读的内存消耗峰值
*   `used_memory_lua`: Lua引擎使用的字节数
*   `mem_fragmentation_ratio`: `used_memory_rss` 与 `used_memory` 的比率
*   `mem_allocator`: 内存分配器，取决于编译时的选择

理想情况下， `used_memory_rss` 的值应该仅仅稍大于 `used_memory` 。
当rss >> used时，很大的不同意味着存在内存碎片（内部或外部），可以通过检查 `mem_fragmentation_ratio` 来评估。
当used >> rss时，意味着部分Redis内存被交换至了操作系统：预示着有明显的延迟。

由于Redis没有内存映射至内存页的控制权，高 `used_memory_rss` 往往是因为内存使用的飙升。

当Redis释放内存时，内存将被交回给分配器，分配器可能会将内存还给操作系统，也可能不会。 `used_memory` 的值和操作系统报告的内存使用量可能存在出入。这可能是因为Redis使用完并释放的内存没有还给操作系统所致。通常可用 `used_memory_peak` 的值来检查此现象。

以下是所有 **persistence** 字段的含义：

*   `loading`: 是否正在装载dump文件的状态值
*   `rdb_changes_since_last_save`: 自上一次dump后改变的数量
*   `rdb_bgsave_in_progress`: 是否正在保存RDB的标志位
*   `rdb_last_save_time`: 上一次成功保存RDB的新纪元时间戳
*   `rdb_last_bgsave_status`: 上一次RDB保存操作的状态值
*   `rdb_last_bgsave_time_sec`: 上一次RDB保存操作经过的秒数
*   `rdb_current_bgsave_time_sec`: 如果正在执行RDB保存操作的话显示已经过的时间
*   `aof_enabled`: 是否激活了AOF日志功能的标志位
*   `aof_rewrite_in_progress`: 是否正在进行AOF重写操作的标志位
*   `aof_rewrite_scheduled`: 是否开启了AOF重写操作的标志位，将在进行中的RDB保存操作完成后进行安排。
*   `aof_last_rewrite_time_sec`: 上一次AOF重写操作经过的秒数
*   `aof_current_rewrite_time_sec`: 如果正在执行AOF重写操作的话显示已经过的时间
*   `aof_last_bgrewrite_status`: 上一次AOF重写操作的状态值

`changes_since_last_save` 表示自上一次调用 `SAVE` 或 `BGSAVE` 后修改数据集的操作数。

如果AOF处于激活状态，将额外显示以下字段：

*   `aof_current_size`: AOF文件当前的大小
*   `aof_base_size`: 最近一次启动或重写AOF文件的大小
*   `aof_pending_rewrite`: 是否开启了AOF重写操作的标志位，将在进行中的RDB保存操作完成后进行安排。
*   `aof_buffer_length`: AOF缓冲区大小
*   `aof_rewrite_buffer_length`:  AOF重写缓冲区大小
*   `aof_pending_bio_fsync`: 在后台I/O队列中的fsync任务等待数量
*   `aof_delayed_fsync`: 延迟的fsync数量

如果正在执行装载操作，将额外显示以下字段：

*   `loading_start_time`: 开始装载时的新纪元时间戳
*   `loading_total_bytes`: 文件大小总和
*   `loading_loaded_bytes`: 已装载的字节数
*   `loading_loaded_perc`: 用比率来表示该值
*   `loading_eta_seconds`: 预计装载完成仍需的秒数

以下是所有 **stats** 字段的含义：

*   `total_connections_received`: 服务器接收的连接总数
*   `total_commands_processed`: 服务器执行的命令总数
*   `instantaneous_ops_per_sec`: 平均每秒执行的命令数
*   `rejected_connections`: 因 `maxclients` 限制导致的拒绝连接数
*   `expired_keys`: 键超时事件总数
*   `evicted_keys`: 因 `maxmemory` 限制而剔除键的数量
*   `keyspace_hits`: 在主词典内键命中的次数
*   `keyspace_misses`: 在主词典内键未找到的次数
*   `pubsub_channels`: 客户端订阅的pub/sub频道的全局数量
*   `pubsub_patterns`: 客户端订阅的pub/sub模式的全局数量
*   `latest_fork_usec`: 最近一次执行分支命令的毫秒数

以下是所有 **replication** 字段的含义：

*   `role`: 如果实例不是从服务器则值为"master"，否则值为"slave"。
    注意从服务器可以作为另一个从服务器的主服务器（雏菊式链接）。

如果实例是从服务器，将额外显示以下字段：

*   `master_host`: 主服务器的Host或IP地址
*   `master_port`: 主服务器的TCP监听端口
*   `master_link_status`: 连接状态（up/down）
*   `master_last_io_seconds_ago`: 自上一次同主服务器交互后经过的秒数
*   `master_sync_in_progress`: 表示主服务器正在与从服务器进行同步

如果正在进行同步操作，将额外显示以下字段：

*   `master_sync_left_bytes`: 距离同步完成剩余的字节数
*   `master_sync_last_io_seconds_ago`: 自上一次在同步操作中进行I/O交换后经过的秒数

如果主从之间的连接宕掉了，将额外显示一个字段：

*   `master_link_down_since_seconds`: 自连接宕掉后经过的秒数

下面的字段总是会显示：

*   `connected_slaves`: 连接的从服务器数量

对于每个从服务器，将添加一行：

*   `slaveXXX`: id，IP地址，端口，状态

以下是所有 **cpu** 字段的含义：

*   `used_cpu_sys`: Redis服务消耗的系统CPU
*   `used_cpu_user`: Redis服务消耗的用户CPU
*   `used_cpu_sys_children`: 后台进程消耗的系统CPU
*   `used_cpu_user_children`: 后台进程消耗的用户CPU

 **commandstats** 提供了基于命令种类的统计，包括调用次数、这些命令消耗CPU的时间总计、以及执行每个命令的平均CPU消耗。

对于每种命令，将添加一行：

*   `cmdstat_XXX`: `calls=XXX,usec=XXX,usec_per_call=XXX`

 **cluster** 现在只包含一个字段：

*   `cluster_enabled`: 表示是否开启了Redis集群

 **keyspace** 提供了每个数据库主目录的统计。统计包括键的数量及过期键的数量。

对于每个数据库，将添加一行：

*   `dbXXX`: `keys=XXX,expires=XXX`

[hcgcpgp]: http://code.google.com/p/google-perftools/

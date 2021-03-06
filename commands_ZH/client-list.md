 `CLIENT LIST` 命令以最具有可读性的格式返回服务器的所有客户端连接及统计信息。

@return

@bulk-string-reply：一个唯一的字符串，使用以下格式：

* 一行显示一个客户端连接（以换行符进行分隔）
* 每行由一连串 属性=值 的格式的字段组成，以空格进行分隔。

每个字段的含义是：

* `id`：一个唯一的64比特的客户端ID（Redis 2.8.12引入）
* `addr`：客户端的地址/端口
* `fd`：套接字对应的文件描述符
* `age`：总计连接时间（秒）
* `idle`：连接空闲时间（秒）
* `flags`：客户端种类（详见下面的说明）
* `db`：所使用数据库的ID
* `sub`：频道订阅数
* `psub`：匹配模式的频道订阅数
* `multi`：处于MULTI/EXEC上下文中的命令数
* `qbuf`：查询缓冲大小（0意味着没有查询缓冲）
* `qbuf-free`：查询缓冲区的空闲容量（0意味着缓冲区已满）
* `obl`：输出缓冲大小
* `oll`：输出列表的长度（当缓冲区满时输出结果被临时存入此队列）
* `omem`：输出缓冲内存使用量
* `events`：文件描述符事件（详见下面的说明）
* `cmd`：最后执行的命令

客户端种类可以为以下某几种的组合：

```
O：客户端为监控模式的从服务实例
S：客户端为普通模式的从服务实例
M：客户端为主服务实例
x：客户端处于MULTI/EXEC上下文中
b：客户端处于阻塞操作等待中
i：客户端处于VM I/O等待中（已弃用）
d：观察的键已被修改 - EXEC将会失败
c：在完成应答的输出后连接将被关闭
u：客户端处于连通状态
U: 客户端以Unix域套接字进行连接
r: 客户端以只读状态访问集群节点
A：客户端将会马上断开连接
N：无特别指定
```

文件描述符事件可以为：

```
r：客户端套接字可读（事件循环）
w：客户端套接字可写（事件循环）
```

## 注意

为了调试，今后的版本会定期添加新的字段。其中一些可能会在将来删除掉。Redis客户端应该充分的考虑到版本兼容性（优雅的处理丢失字段、跳过未知字段）。

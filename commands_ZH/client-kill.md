 `CLIENT KILL` 命令用来关闭指定的客户端连接。
Redis 2.8.11及以前版本可以关闭指定客户端地址的连接，格式为：

    CLIENT KILL addr:port

`ip:port` 应匹配命令 `CLIENT LIST` 返回值的其中一行（ `addr` 字段）。

Redis从 2.8.12 开始，命令接受新的格式：

    CLIENT KILL <filter> <value> ... ... <filter> <value>

新格式可以按属性而不只是通过地址关闭客户端。
可用属性包括：

* `CLIENT KILL ADDR ip:port` 。这和旧的三个参数的版本行为相同。 
* `CLIENT KILL ID client-id`. Allows to kill a client by its unique `ID` field, which was introduced in the `CLIENT LIST` command starting from Redis 2.8.12.
* `CLIENT KILL ID client-id` 。根据表示唯一性的字段 `ID` 关闭客户端，自Redis 2.8.12 引入， `CLIENT LIST` 命令也是。
* `CLIENT KILL TYPE type` ， *type* 取值为 `normal` 、 `slave` 或 `pubsub` 。 
关闭指定类型的 **所有客户端** 连接。 
注意正在执行 `MONITOR` 命令的客户端类型为 `normal` 。
* `CLIENT KILL SKIPME yes/no` 。 
默认选项为 `yes` ，即执行此命令的客户端不会被关闭，而使用 `no` 的话当前执行的客户端的连接也将会被关闭。

可以同时提供多个属性。命令将会以与的关系合并多个属性条件。例如：

    CLIENT KILL addr 127.0.0.1:6379 type slave

这个命令将会关闭指定地址的从服务连接。目前这种多属性的方式用的还比较少。

使用新格式的命令不再返回 `OK` 或错误，而是返回关闭客户端的数量，可能会是零。

## CLIENT KILL 与 Redis Sentinel

近期版本的 Redis Sentinel（Redis 2.8.12 或更新）使用CLIENT KILL断开重新配置了的实例连接，以迫使客户端重新进行Sentinel握手及更新配置。

## 注意

由于Redis的单线程特性，客户端正在执行命令的时候是无法杀掉其连接的。从客户端的角度来看，执行命令的过程中是不会被断开连接的。然而，当再执行下一个命令时就会得知连接已被关闭（得到网络错误信息）。

@return

@simple-string-reply：如果连接存在并且成功关闭返回 `OK`

当使用属性/值的格式调用时：

@integer-reply: 关闭客户端的数量。
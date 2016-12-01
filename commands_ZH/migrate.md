原子性的将键从源Redis实例迁移至目标Redis实例。
当键从源实例中删除并保证存在于目标实例时宣告操作成功。

此命令具有原子性，在传递键的时候需要阻塞住两个实例，其中任一时刻键要么出现在目标实例，要么出现在源实例，除非发生了超时错误。
在3.2及以上版本，可以以流水线方式通过一次调用迁移多个键，方式为传递空字符串作为键，并且添加 `KEYS` 子句。

此命令在内部使用 `DUMP` 生成序列化版本的键值，然后在目标实例中使用 `RESTORE` 将键进行反序列化。
此时源实例为目标实例的一个客户端。
如果目标实例执行 `RESTORE` 命令返回OK，源实例将使用 `DEL` 删除键。

超时参数是与目标实例通信的最大停顿时间，单位为毫秒。
这意味着操作并不需要在指定毫秒间完成，而是要在指定毫秒间完成传输，从而不再阻塞实例。

`MIGRATE` 需要执行 I/O 操作并承诺超时时间。
当在传输时出现 I/O 错误或到达超时时间，操作将终止并返回指定错误 - `IOERR` 。
当发生此现象时可能发生以下两种情况：

* 键将存在于两个实例。
* 键仅存在于源实例。

在超时发生的情况下键不可能丢失，但调用 `MIGRATE` 的客户端遇到超时错误时应检查键是否 _也_ 存在于目标实例中，并采取相应的对策。

当发生其他错误（以 `ERR` 开头）时 `MIGRATE` 担保键值只存在于源实例中（除非目标实例也 _已经_ 存在一个相同名称的键）。

如果源实例中没有可迁移的键，返回 `NOKEY` 。
通常情况下是有可能出现未命中的键的，比如过期的键，这时 `NOKEY` 并不代表出错。

## 一次迁移多个键

从Redis 3.0.6开始 `MIGRATE` 支持一个新的块传输模式，使用管道在实例间迁移多个键，用来改进多次调用 `MIGRATE` 迁移单个键导致的往返延迟和其他开销。

此功能通过使用选项 `KEYS` 和空的参数 *键* 实现。
真正的键名放在参数 `KEYS` 之后，如下面这个例子：

    MIGRATE 192.168.1.34 6379 "" 0 5000 KEYS key1 key2 key3

当实例中没有存在的键时返回 `NOKEY` 状态码，否则命令将会执行，即使只有一个键存在。

## 选项

* `COPY` -- 不从本地实例中删除键。
* `REPLACE` -- 替换目标实例中已存在的键。
* `KEYS` -- 如果参数键是空字符串，命令将会迁移选项 `KEYS` 之后的所有键（更多信息见上面的段落）。

`COPY` 和 `REPLACE` 只在3.0及以上版本中可用。
`KEYS` 从3.0.6版本开始可用。

@return

@simple-string-reply: 命令执行成功时返回OK。如果没有在源实例中找到键则返回 `NOKEY` 。
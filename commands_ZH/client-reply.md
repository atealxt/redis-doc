对客户端来说有时候需要禁掉Redis服务器的应答。
比如当客户端发送即发即弃型命令、执行大量数据加载，或在缓存环境中不断刷新数据流。
在这些情况下使用服务器的时间及带宽来应答客户端是一种浪费，应该被忽略掉。

命令 `CLIENT REPLY` 用来控制是否让服务器对客户端命令进行应答。
共有以下模式：

* `ON`. 默认模式，服务器对所有命令进行应答。
* `OFF`. 此模式下服务器不对客户端命令进行应答。
* `SKIP`. 此模式下会立即跳过它之后的命令应答。

@return

当使用 `OFF` 或 `SKIP` 调用命令时，没有应答。
当使用 `ON` 调用命令时，返回：

@simple-string-reply: `OK`.

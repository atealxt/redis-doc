禁止客户端访问Redis集群的从服务节点。

这是默认设置，可以使用命令 `READONLY` 进行修改。
命令 `READWRITE` 用来将当前连接的只读模式标志位重置为读写模式。

@return

@simple-string-reply

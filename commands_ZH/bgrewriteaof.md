通知Redis开启一个重写[只可追加的文件][tpaof]的进程。
重写进程会创建一个比当前的只可追加的文件有少许优化的版本。

[tpaof]: /topics/persistence#append-only-file

如果 `BGREWRITEAOF` 执行失败，旧的AOF将不变并且不会有数据丢失。

重写只会在当前没有后台进程进行持久化操作的时候由Redis触发。
特别说明：

* 如果一个Redis子进程正在磁盘上创建镜像，在创建RDB文件结束之前AOF重写将处于 _预订_ 状态且不会执行。
  在这种情况下 `BGREWRITEAOF` 仍然会返回OK状态码，但是会附带一个适当的消息。
  可以使用Redis 2.6版本的INFO命令，手工确认AOF重写是否已经预订。
* 如果正在执行AOF重写，命令将会返回错误并且不会再一次安排执行。

从Redis 2.4版本以后AOF重写由Redis自动触发，但仍然可以在任何时间用 `BGREWRITEAOF` 命令进行手动触发。

详细信息请参见[持久化文档][tp]。

[tp]: /topics/persistence

@return

@simple-string-reply: 总是 `OK` 。

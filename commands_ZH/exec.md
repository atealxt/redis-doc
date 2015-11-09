在 [事务][tt] 中执行所有之前排队的命令，并且恢复至普通连接状态。

[tt]: /topics/transactions

当使用 `WATCH` 时， `EXEC` 命令只在被观察的键没有被修改的时候执行，即[check-and-set原理][ttc]。

[ttc]: /topics/transactions#cas

@return

@array-reply: 在原子事务中的所有命令的应答。

当使用 `WATCH` 时，如果执行被终止 `EXEC` 将返回@nil-reply。

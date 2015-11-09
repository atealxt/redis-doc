清除在 [事务][tt] 中所有之前排队的命令并恢复为普通连接状态。

[tt]: /topics/transactions

如果之前使用过 `WATCH` 命令， `DISCARD` 将取消该连接所监视的所有键。

@return

@simple-string-reply: 总是 `OK` 。

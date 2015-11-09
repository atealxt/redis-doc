标记一段 [事务][tt] 的开始。
随后的命令将被存入队列并以 `EXEC` 进行原子性执行。

[tt]: /topics/transactions

@return

@simple-string-reply: 总是 `OK` 。

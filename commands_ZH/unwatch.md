清除所有之前[事务][tt]中监视的键。

[tt]: /topics/transactions

如果调用了 `EXEC` 或 `DISCARD` ，就没有必要再手动调用 `UNWATCH` 了。

@return

@simple-string-reply: 总是 `OK` 。

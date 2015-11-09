`BRPOPLPUSH` 是阻塞版本的 `RPOPLPUSH` 。
当 `源` 列表包含元素时，此命令的行为与 `RPOPLPUSH` 完全相同。
当在 `MULTI`/`EXEC` 中使用时，行为与 `RPOPLPUSH` 一样。
当 `源` 列表为空时，Redis会阻塞住连接直到有另外的客户端向此列表放入元素或到达 `超时` 时间。
`超时` 设为零可以用作无期限阻塞。

更多信息请参见 `RPOPLPUSH` 。

@return

@bulk-string-reply: 元素从 `源` 列表弹出并推入至  `目标` 列表。
如果到达 `超时` ，返回@nil-reply。

## 模式：可靠性队列

有关此模式的描述请参见 `RPOPLPUSH` 文档。

## 模式：环形列表

有关此模式的描述请参见 `RPOPLPUSH` 文档。

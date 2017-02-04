删除当前所选数据库中的所有键。
此命令永远不会执行失败。

此操作的时间复杂度为 O(N)，N是数据库的键的数量。

`FLUSHDB ASYNC` （Redis 4.0.0 及以上版本）
---
文档见 `FLUSHALL` 。

@return

@simple-string-reply

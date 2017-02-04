删除所有存在的数据库中的所有键，而不仅仅是当前所选的。
此命令永远不会执行失败。

此操作的时间复杂度为 O(N)，N是所有数据库的键的数量。

`FLUSHALL ASYNC` （Redis 4.0.0 及以上版本）
---
Redis如今可以在后台线程删除键，而不会阻塞整个服务。
`FLUSHALL` 和 `FLUSHDB` 增加了选项 `ASYNC` ，可让一个数据库或整个数据集实现异步。

@return

@simple-string-reply

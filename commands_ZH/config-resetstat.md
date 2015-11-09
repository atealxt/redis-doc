重置由Redis使用 `INFO` 命令报告的统计数据。

以下这些计数器会被重置：

* 键命中数
* 键未命中数
* 执行命令数
* 接受的连接数
* 超时的键数
* 拒绝的连接数
* 最近派生(2)时间
* `aof_delayed_fsync` 数

@return

@simple-string-reply: 总是 `OK` 。

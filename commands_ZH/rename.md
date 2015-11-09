将 `键` 重命名为 `新键` 。
当原键名和目标键名相同、或 `键` 不存在时返回错误。
如果 `新键` 已存在，值将被重写。此时 `RENAME` 会内部执行一次 `DEL` 操作，所以当删除的键所包含的值很大时有可能会造成较长的延迟，即使 `RENAME` 本身通常是一个常量时间的操作。

@return

@simple-string-reply

@examples

```cli
SET mykey "Hello"
RENAME mykey myotherkey
GET myotherkey
```

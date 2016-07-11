将 `键` 重命名为 `新键` 。
当 `键` 不存在时返回错误。
如果 `新键` 已存在，值将被重写，此时 `RENAME` 将内部执行一个 `DEL` 操作，所以当删除的键所包含的值很大时有可能会造成较长的延迟，即使 `RENAME` 本身通常是一个常量时间的操作。

**注意：** 在Redis 3.2.0 版本之前如果源键和目标键名称相同将返回错误。

@return

@simple-string-reply

@examples

```cli
SET mykey "Hello"
RENAME mykey myotherkey
GET myotherkey
```

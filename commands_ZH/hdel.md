删除存储在 `键` 的哈希表中的指定字段。
如果指定字段不存在则忽略。
如果 `键` 不存在，将以空哈希表进行处理并返回 `0` 。
@return

@integer-reply: 从哈希表移除的字段数，不包括指定但不存在的字段。

@history

*   `>= 2.4`: 接受多个 `字段` 参数。
    早于2.4版本的Redis只能在每次调用时删除一个字段。

    如果要在早些版本中原子性的从哈希表中删除多个字段，请使用 `MULTI` / `EXEC` 代码块。

@examples

```cli
HSET myhash field1 "foo"
HDEL myhash field1
HDEL myhash field2
```

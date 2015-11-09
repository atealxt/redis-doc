返回存储在 `键` 内哈希表中指定字段的值。

每当在哈希表中遇到 `字段` 不存在时，返回 `nil` 值。
由于不存在的键被当作空的哈希表进行处理，所以对不存在的 `键` 执行 `HMGET` 将会返回包含一串 `nil` 值的列表。

@return

@array-reply: 以请求时相同的顺序，返回给定字段对应值的列表。

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HMGET myhash field1 field2 nofield
```

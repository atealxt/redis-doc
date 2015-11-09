返回存储在 `键` 中的哈希表包含的字段数量。

@return

@integer-reply: 返回哈希表中的字段数量，或当 `键` 不存在时返回 `0` 。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HLEN myhash
```

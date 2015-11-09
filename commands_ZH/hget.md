返回指定 `键` 存储的哈希表中指定 `字段` 的值。

@return

@bulk-string-reply: 返回`字段` 中的值，或当 `字段` 不存在于哈希表内或 `键` 不存在的时候返回 `nil` 。

@examples

```cli
HSET myhash field1 "foo"
HGET myhash field1
HGET myhash field2
```

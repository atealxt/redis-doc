返回存储在 `键` 中的哈希表内所有字段的名称。

@return

@array-reply: 返回哈希表中的字段列表，或者当 `键` 不存在时返回空列表。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HKEYS myhash
```

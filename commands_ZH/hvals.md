返回存储在 `键` 内哈希表中的所有值。

@return

@array-reply: 返回哈希表值的列表，或当 `键` 不存在时返回空列表。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HVALS myhash
```

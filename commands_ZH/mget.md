返回所有指定键的值。
对于任何不包含字符串值或不存在的键，返回 `nil` 值。
因此，本操作不会执行失败。

@return

@array-reply: 指定键的值列表。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
MGET key1 key2 nonexisting
```

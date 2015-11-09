返回存储在指定 `键` 的所有字段和值。
在返回值中，每个字段名的下一位是它的值，所以返回值的长度是哈希表长度的两倍。

@return

@array-reply: 返回存储在哈希表中的字段与值的列表，或当 `键` 不存在时返回空列表。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HGETALL myhash
```

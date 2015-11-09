删除指定的键。
如果键不存在则忽略。

@return

@integer-reply: 删除的键的数量。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
DEL key1 key2 key3
```

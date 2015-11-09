此命令等同于 `SINTER` ，除了以存储到 `目标` 取代了直接返回结果集合。

如果  `目标` 已存在，将被重写。

@return

@integer-reply: 结果集合的元素数量。

@examples

```cli
SADD key1 "a"
SADD key1 "b"
SADD key1 "c"
SADD key2 "c"
SADD key2 "d"
SADD key2 "e"
SINTERSTORE key key1 key2
SMEMBERS key
```

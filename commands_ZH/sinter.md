返回所有给定集合的交集。

例如：

```
key1 = {a,b,c,d}
key2 = {c}
key3 = {a,c,e}
SINTER key1 key2 key3 = {c}
```

不存在的键会被当作空集合。
如果有的键的值是空集合，结果集合将也为空（与空集合的交集永远为空集合）。

@return

@array-reply: 结果集合的列表。

@examples

```cli
SADD key1 "a"
SADD key1 "b"
SADD key1 "c"
SADD key2 "c"
SADD key2 "d"
SADD key2 "e"
SINTER key1 key2
```

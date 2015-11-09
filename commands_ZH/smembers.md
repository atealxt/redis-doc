返回所有存储在 `键` 的集合的成员。

这和运行 `SINTER` 带一个参数 `键` 具有相同的效果。

@return

@array-reply: 集合的所有元素。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SMEMBERS myset
```

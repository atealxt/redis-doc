返回存储在 `键` 的集合的势（元素数量）。

@return

@integer-reply: 返回集合的势（元素数量），或当 `键` 不存在时返回 `0` 。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SCARD myset
```

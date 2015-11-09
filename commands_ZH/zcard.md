返回存储在 `键` 的有序集合的基数（元素数量）。

@return

@integer-reply: 返回有序集合的基数（元素数量），或当 `键` 不存在时返回 `0` 。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZCARD myzset
```

返回存储在 `键` 的有序集合中指定范围的元素。
元素将按照分数从高到低进行排序。
对于相同的分数采用倒序词典排序。

除了采用倒序排序之外， `ZREVRANGE` 与 `ZRANGE` 类似。

@return

@array-reply: 指定范围的元素列表（也可以伴随着它们的分数）。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANGE myzset 0 -1
ZREVRANGE myzset 2 3
ZREVRANGE myzset -2 -1
```

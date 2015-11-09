返回存储在 `键` 的有序集合中分数处于 `最大值` 与 `最小值` 之间的所有元素（包含分数等于 `最大值` 或 `最小值` 的元素）。
与有序集合的默认排序相反，此命令元素将按照分数从高到低进行排序。

相同分数的元素将进行倒序词典排序后返回。

除了倒序排序之外， `ZREVRANGEBYSCORE` 与 `ZRANGEBYSCORE` 类似。

@return

@array-reply: 指定分数区间的元素列表（也可以伴随着它们的分数）。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANGEBYSCORE myzset +inf -inf
ZREVRANGEBYSCORE myzset 2 1
ZREVRANGEBYSCORE myzset 2 (1
ZREVRANGEBYSCORE myzset (2 (1
```

从存储在 `键` 的有序集合中移除所有得分处于 `最小值` 与 `最大值` 之间的元素（闭区间）。

从2.1.6版本开始，可以不包含 `最小值` 与 `最大值` ，句法参见 `ZRANGEBYSCORE` 。

@return

@integer-reply: 移除的元素数量。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREMRANGEBYSCORE myzset -inf (2
ZRANGE myzset 0 -1 WITHSCORES
```

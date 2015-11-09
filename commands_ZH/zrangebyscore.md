返回存储在 `键` 的有序集合中分数处于 `最小值` 与 `最大值` 之间的所有元素（包含分数等于 `最小值` 或 `最大值` 的元素）。
元素将按照分数从低到高进行排序。

相同分数的元素将进行词典排序后返回（遵循了Redis的有序集合的实现，不会引起多余的计算）。

可选参数 `LIMIT` 可用来选取指定范围内的匹配元素（类似于SQL中的 _SELECT LIMIT offset, count_ ）。
请记住如果 `offset` 很大，在取得返回元素之前有序集合需要跨越 `offset` 个元素，增加了O(N)的时间复杂度。

可选参数 `WITHSCORES` 让命令同时返回元素与参数，而不单单是元素。
此选项从Redis 2.0版本开始可用。

## 区间限制

 `最小值` 与 `最大值` 可以为 `-inf` 和 `+inf` ，这样就可以在不必知道有序集合的最高或最低分数的情况下取得所有元素。

默认情况下， `最小值` 与 `最大值` 为闭区间（包含）。
可以给分数加字符前缀 `(` 指定其为开区间（不包含）。
例如：

```
ZRANGEBYSCORE zset (1 5
```

将返回 `1 < 分数 <= 5` 的所有元素，而：

```
ZRANGEBYSCORE zset (5 (10
```

将返回 `5 < 分数 < 10` 的所有元素（不包含5和10）。

@return

@array-reply: 指定分数区间的元素列表（也可以伴随着它们的分数）。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZRANGEBYSCORE myzset -inf +inf
ZRANGEBYSCORE myzset 1 2
ZRANGEBYSCORE myzset (1 2
ZRANGEBYSCORE myzset (1 (2
```

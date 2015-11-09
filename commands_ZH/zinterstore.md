计算指定 `键数` 的有序集合的交集，并将结果存至 `目标` 中。
必须在传递输入键和其他（可选）参数的前面提供输入键的个数的参数（ `键数` ）。

默认下，结果集元素的分数是所有有序集合元素分数的和。
由于交集要求元素存在于每个给定的有序集合内，结果集中每个元素的分数等于所有输入有序集合其元素分数的和。

关于 `权重` 与 `聚集` 选项的描述，参见 `ZUNIONSTORE` 。

如果 `目标` 已存在，将被重写。

@return

@integer-reply: 存储在 `目标` 的有序结果集合的元素数量。

@examples

```cli
ZADD zset1 1 "one"
ZADD zset1 2 "two"
ZADD zset2 1 "one"
ZADD zset2 2 "two"
ZADD zset2 3 "three"
ZINTERSTORE out 2 zset1 zset2 WEIGHTS 2 3
ZRANGE out 0 -1 WITHSCORES
```

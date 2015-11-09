从存储在 `键` 的有序集合中移除所有排名处于 `起始` 与 `结束` 之间的元素。
 `起始` 与 `结束` 皆为以 `0` 开始的索引，排名为 `0` 的元素分数最低。
索引可以为负数，即从最高得分的元素开始索引。
例如： `-1` 为最高得分的元素、 `-2` 为第二高得分的元素，以此类推。

@return

@integer-reply: 移除的元素数量。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREMRANGEBYRANK myzset 0 1
ZRANGE myzset 0 -1 WITHSCORES
```

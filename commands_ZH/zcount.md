返回 `键` 的有序集合中分数处于 `最小` 与 `最大` 之间的元素数量。

 `最小` 与 `最大` 参数的语义与 `ZRANGEBYSCORE` 中的描述相同。

注意：此命令的复杂度仅为 O(log(N)) 因为它使用了元素排名（见 `ZRANK` ）来获取范围。
因此不需要再计算范围的大小。

@return

@integer-reply: 在指定分数区间内的元素数量。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZCOUNT myzset -inf +inf
ZCOUNT myzset (1 3
```

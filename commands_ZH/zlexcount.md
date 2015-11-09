当有序集合中所有元素的分数相同时，为了保持字典顺序，此命令返回有序集合中 `键` 的值从 `最小` 至 `最大` 之间的元素数量。

参数 `最小` 和 `最大` 的含义和 `ZRANGEBYLEX` 的一样。

注意：此命令的复杂度仅为 O(log(N)) 因为它使用了元素排名（见 `ZRANK` ）来获取范围。
因此不需要再计算范围的大小。

@return

@integer-reply: 指定分数区间内的元素数量。

@examples

```cli
ZADD myzset 0 a 0 b 0 c 0 d 0 e
ZADD myzset 0 f 0 g
ZLEXCOUNT myzset - +
ZLEXCOUNT myzset [b [f
```

When all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering, this command returns all the elements in the sorted set at `key` with a value between `max` and `min`.
当有序集合中所有元素的分数相同时，为了保持字典顺序，此命令返回有序集合中 `键` 的值从 `最大` 至 `最小` 之间的所有元素。

除了排序是反向的， `ZREVRANGEBYLEX` 与  `ZRANGEBYLEX` 相同。

@return

@array-reply: 指定分数区间内的元素。

@examples

```cli
ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
ZREVRANGEBYLEX myzset [c -
ZREVRANGEBYLEX myzset (c -
ZREVRANGEBYLEX myzset (g [aaa
```

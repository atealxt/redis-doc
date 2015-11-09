当有序集合中所有元素的分数相同时，为了保持字典顺序，此命令删除有序集合中 `键` 的值从 `最小` 至 `最大` 之间的元素。

参数 `最小` 和 `最大` 的含义和 `ZRANGEBYLEX` 的一样。
同样地，当参数相同时作用于此命令的元素与 `ZRANGEBYLEX` 的相同。

@return

@integer-reply: 删除的元素数量。

@examples

```cli
ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
ZRANGE myzset 0 -1
ZREMRANGEBYLEX myzset [alpha [omega
ZRANGE myzset 0 -1
```

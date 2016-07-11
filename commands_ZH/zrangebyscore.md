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

## 模式：元素的加权随机选择

通常 `ZRANGEBYSCORE` 拿来用于取得范围内的元素，分数作为其整数索引键。

而它也可以用来做一些其他的事，一些常见的问题比如实现马尔可夫链或其他需要从一个集合中随机选择元素，但不同元素有着不同的选择可能性权重的算法。

使用此命令实现这样一个算法：

假设你有元素A、B和C，权重分别是1、2和3。
权重和为1+2+3 = 6。

使用这个算法将所有元素加到有序集合中：

```
SUM = ELEMENTS.TOTAL_WEIGHT // 在这里值是6。
SCORE = 0
FOREACH ELE in ELEMENTS
    SCORE += ELE.weight / SUM
    ZADD KEY SCORE ELE
END
```

结果集合为：

```
A的分数为0.16
B的分数为.5
C的分数为1
```

由于计算是取近似值，为了避免C的值不是1而是比如0.998，这里修改了算法，确保最后一个分数为1（具体实现留给读者作为练习。。。）。

这样，想要取得加权随机元素，每次只需要计算一个0到1之间的随机数（比如大多数语言的 `rand()` ），如：

    RANDOM_ELE = ZRANGEBYSCORE key RAND() +inf LIMIT 0 1

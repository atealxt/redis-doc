从存储在 `键` 的有序集合中移除指定的成员。
不存在的成员将被忽略。

当 `键` 存在但值不是有序集合时返回错误。

@return

@integer-reply, 具体为：

* 从有序集合中移除成员的数量，不包括不存在的成员。

@history

* `>= 2.4`: 接受多个元素。
  早于2.4版本的Redis只可以一次移除一个成员。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREM myzset "two"
ZRANGE myzset 0 -1 WITHSCORES
```

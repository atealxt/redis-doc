返回存储在 `键` 的有序集合中的 `成员` 的排名，排名以分数从高到低进行排列。
排名（或称索引）以零开始，最高分数的成员的排名为 `0` 。

使用 `ZRANK` 可以取得元素的排名以分数从低到高进行排列。

@return

* 如果 `成员` 存在于有序集合中， @integer-reply:  `成员` 的排名。
* 如果 `成员` 不存在于有序集合中或 `键` 不存在， @bulk-string-reply: `nil` 。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANK myzset "one"
ZREVRANK myzset "four"
```

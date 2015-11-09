对存储在 `键` 的有序集合中 `成员` 的分数进行自增 `自增数` 。
如果 `成员` 不存在于有序集合中，将把其加入并设置分数为 `增加数` （就像之前的分数如 `0.0` 一样）。
如果 `键` 不存在，将创建一个新的只包含指定 `成员` 的有序集合。

如果 `键` 存在但值的类型不是有序集合时返回错误。

分数的值应为一个数字字符串，解释为双精度浮点数。
可以提供负数来对分数进行递减。

@return

@bulk-string-reply: `成员` 的新分数（双精度浮点数），以字符串表示。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZINCRBY myzset 2 "one"
ZRANGE myzset 0 -1 WITHSCORES
```

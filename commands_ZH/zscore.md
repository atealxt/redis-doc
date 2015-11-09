返回存储在 `键` 的有序集合中 `成员` 的分数。

如果 `成员` 不存在于有序集合中，或 `键` 不存在，返回 `nil` 。

@return

@bulk-string-reply:  `成员` 的分数（双精度浮点数），以字符串形式表示。

@examples

```cli
ZADD myzset 1 "one"
ZSCORE myzset "one"
```

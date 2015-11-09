从存储在 `键` 的集合中删除指定成员。
如果指定的成员不在集合中则忽略。
如果 `键` 不存在，将作为空集合处理，命令返回 `0` 。

如果存储在 `键` 的值类型不是集合将返回错误。

@return

@integer-reply: 从集合中删除的成员数量，不包括不存在的成员。

@history

* `>= 2.4`: 接受多个 `成员` 参数。
  早于2.4版本的Redis只能一次删除一个集合成员。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myset "three"
SREM myset "one"
SREM myset "four"
SMEMBERS myset
```

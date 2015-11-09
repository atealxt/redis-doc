向存储在 `键` 内的集合添加指定的成员。
已存在于集合中的指定成员将被忽略。
如果 `键` 不存在，在添加指定成员前会创建一个新的集合。

如果存储在 `键` 内的值类型不是集合将返回错误。

@return

@integer-reply: 添加到集合内的元素，不包括所有已存在于集合中的元素。

@history

* `>= 2.4`: 接受多个 `成员` 参数。
 早于2.4版本的Redis只可以每次添加一个成员。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SADD myset "World"
SMEMBERS myset
```

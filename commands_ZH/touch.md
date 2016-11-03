更新键的最后访问时间。
键不存在则忽略。

@return

@integer-reply: 更新了的键的数量。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
TOUCH key1 key2
```

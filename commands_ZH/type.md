返回存储在 `键` 的值类型的字符串描述。
可供返回的类型有：`string` 、 `list` 、 `set` 、 `zset` 和 `hash` 。

@return

@simple-string-reply: 返回 `键` 的值类型，或当 `键` 不存在时返回 `none` 。

@examples

```cli
SET key1 "value"
LPUSH key2 "value"
SADD key3 "value"
TYPE key1
TYPE key2
TYPE key3
```

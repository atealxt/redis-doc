返回存储在 `键` 的字符串长度。
如果 `键` 持有的是非字符串值则返回错误。

@return

@integer-reply: 返回存储在 `键` 的字符串长度，或当 `键` 不存在时返回 `0` 。

@examples

```cli
SET mykey "Hello world"
STRLEN mykey
STRLEN nonexisting
```

取得 `键` 的值。
如果键不存在则返回特殊值 `nil` 。
如果 `键` 存储的值不是字符串类型则返回错误，因为 `GET` 只处理字符串。

@return

@bulk-string-reply： `键` 的值，或当 `键` 不存在时返回 `nil` 。

@examples

```cli
GET nonexisting
SET mykey "Hello"
GET mykey
```

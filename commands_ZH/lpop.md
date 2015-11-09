移除并返回存储在 `键` 内列表的第一个元素。

@return

@bulk-string-reply: 返回第一个元素的值，或当 `键` 不存在时返回 `nil` 。

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LPOP mylist
LRANGE mylist 0 -1
```

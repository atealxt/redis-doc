移除并返回储存于 `键` 的列表中的最后一个元素。

@return

@bulk-string-reply: 返回最后一个元素的值，或当 `键` 不存在时返回 `nil` 。

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
RPOP mylist
LRANGE mylist 0 -1
```

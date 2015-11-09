将列表 `index` 处的值设置为 `value` 。
关于 `index` 的更多信息，请参见 `LINDEX` 。

超出范围的索引将返回错误。

@return

@simple-string-reply

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LSET mylist 0 "four"
LSET mylist -2 "five"
LRANGE mylist 0 -1
```

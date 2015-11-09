返回存储在 `键` 的列表索引 `index` 处的元素。
索引以零开始，所以 `0` 意味着第一个元素， `1` 意味着第二个元素，以此类推。
可以使用负数索引从列表末尾选取元素。
此处 `-1` 意味着最后一个元素， `-2` 意味着倒数第二个元素，以此类推。

当 `键` 的值不是列表时返回错误。

@return

@bulk-string-reply: 返回请求的元素，或当 `index` 超出索引范围时返回 `nil` 。

@examples

```cli
LPUSH mylist "World"
LPUSH mylist "Hello"
LINDEX mylist 0
LINDEX mylist -1
LINDEX mylist 3
```

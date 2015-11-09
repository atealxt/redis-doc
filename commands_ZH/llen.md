返回存储在 `键` 的列表长度。
如果 `键` 不存在，则当作一个空列表并返回 `0` 。
当存储在 `键` 的值的类型不是列表时返回错误。

@return

@integer-reply: 存储在 `键` 的列表的长度。

@examples

```cli
LPUSH mylist "World"
LPUSH mylist "Hello"
LLEN mylist
```

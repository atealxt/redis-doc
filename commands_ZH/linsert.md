向存储在 `键` 的列表中的参照 `支点` 值的前面或后面插入 `值` 。

如果 `键` 不存在，则当作一个空列表，不作任何操作。

当 `键` 存在但值类型不是列表时返回错误。

@return

@integer-reply: 返回插入后的列表长度，或当没有找到 `支点` 值时返回 `-1` 。

@examples

```cli
RPUSH mylist "Hello"
RPUSH mylist "World"
LINSERT mylist BEFORE "World" "There"
LRANGE mylist 0 -1
```

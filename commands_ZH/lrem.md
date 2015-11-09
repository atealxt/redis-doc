从存储在 `键` 的列表中删除前 `数量` 个碰到的与 `指定值` 等值的元素。
 `数量` 参数影响操作有以下几种情况：

* `count > 0`: 从表头至表尾删除值为 `指定值` 的元素。
* `count < 0`: 从表尾至表头删除值为 `指定值` 的元素。
* `count = 0`: 删除所有值为 `指定值` 的元素。

例如， `LREM list -2 "hello"` 将删除存储在 `list` 的列表中后2个值为 `"hello"` 的元素。

注意不存在的键将作为空列表进行处理，所以当 `键` 不存在时，命令将总是返回 `0` 。

@return

@integer-reply: 移除的元素数量。

@examples

```cli
RPUSH mylist "hello"
RPUSH mylist "hello"
RPUSH mylist "foo"
RPUSH mylist "hello"
LREM mylist -2 "hello"
LRANGE mylist 0 -1
```

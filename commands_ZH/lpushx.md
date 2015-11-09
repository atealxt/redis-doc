将 `值` 插入存储在 `键` 的列表头部，当且仅当 `键` 存在并且值类型为列表的时候。
与 `LPUSH` 不同，当 `键` 不存在时将不会执行任何操作。

@return

@integer-reply: 压栈操作后的列表长度。

@examples

```cli
LPUSH mylist "World"
LPUSHX mylist "Hello"
LPUSHX myotherlist "Hello"
LRANGE mylist 0 -1
LRANGE myotherlist 0 -1
```

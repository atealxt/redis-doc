向存储在 `键` 的列表末尾插入所有指定的值。
如果 `键` 不存在，在推入操作前会创建一个空列表。
如果 `键` 持有值的类型不是列表将返回错误。

可以使用单个命令推入多个元素，仅仅通过在命令后面指定多个参数。
从最左面到最右面，元素被一个个的插入到列表尾部。
例如命令 `RPUSH mylist a b c` 的结果为列表的首元素是 `a` ，第二个元素是 `b` ，第三个元素是 `c` 。

@return

@integer-reply: 插入操作后的列表长度。

@history

* `>= 2.4`: 接受多个 `值` 参数。
  早于2.4版本的Redis只可以每次插入一个值。

@examples

```cli
RPUSH mylist "hello"
RPUSH mylist "world"
LRANGE mylist 0 -1
```

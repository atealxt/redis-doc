向存储在 `键` 内的列表头部插入所有指定的值。
如果 `键` 不存在，在执行压栈操作前会创建一个空列表。
当 `键` 所对应的值不是列表时返回错误。

可以仅使用一次命令并在命令后面传入多个参数来插入多个元素。
元素将被一个个的插入列表头部，从最左面的元素到最右面的元素。
所以例如命令 `LPUSH mylist a b c` 将会创造一个首元素是 `c` 、次元素是 `b` 、第三个元素是 `a` 的列表。

@return

@integer-reply: 压栈操作后的列表长度。

@history

* `>= 2.4`: 接受多个 `value` 参数。
  在早于2.4版本的Redis中只可以一次插入一个值。

@examples

```cli
LPUSH mylist "world"
LPUSH mylist "hello"
LRANGE mylist 0 -1
```

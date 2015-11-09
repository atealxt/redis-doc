对 `键` 进行原子性设 `值` 并返回 `键` 中存储的旧值。
当 `键` 存在但值不是字符串的时候返回错误。

## 设计模式

`GETSET` 可以和 `INCR` 一起使用，用来实现带有原子性复位的计数器。
例如：进程可以在发生一些事件的时候对键 `mycounter` 调用 `INCR` 命令，但是有时候我们需要取得计数器的值并原子性的把它重置为零。
这个需求可以使用 `GETSET mycounter "0"` 来办到：

```cli
INCR mycounter
GETSET mycounter "0"
GET mycounter
```

@return

@bulk-string-reply: 存储于 `键` 的旧值，或当 `键` 不存在时返回 `nil` 。

@examples

```cli
SET mykey "Hello"
GETSET mykey "World"
GET mykey
```

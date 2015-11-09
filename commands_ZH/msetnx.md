对给定的键分别设置相应的值。
如果有键已经存在，即使只有一个， `MSETNX` 也不会执行任何操作。

因此 `MSETNX` 可以用于给唯一性逻辑对象的不同的键设置不同的值，以确保所有字段要么都被设值要么都不被设值。

`MSETNX` 为原子性，所以给定的所有键都将同时进行设置。
客户端不可能遇到有些键被更新了而其他键没有改变的情况。

@return

@integer-reply, 值为：

* `1` 如果成功给所有键设值。
* `0` 如果没有给键设值（至少有一个已存在的键）。

@examples

```cli
MSETNX key1 "Hello" key2 "there"
MSETNX key2 "there" key3 "world"
MGET key1 key2 key3
```

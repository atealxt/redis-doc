为存储在 `键` 中哈希表内的指定字段设值，当且仅当 `字段` 不存在的时候。
如果 `键` 不存在，将会创建一个新的持有哈希表的键。
如果 `字段` 已经存在于哈希表内，则此操作没有任何效果。

@return

@integer-reply, 具体为：

* 如果 `字段` 在哈希表中为新的字段并且成功设置了 `值` 则返回 `1` 。
* 如果 `字段` 在哈希表中已经存在则不做任何操作并返回 `0` 。

@examples

```cli
HSETNX myhash field "Hello"
HSETNX myhash field "World"
HGET myhash field
```

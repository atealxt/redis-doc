让存储在 `键` 的哈希表中 `字段` 的数进行自增 `自增数` 操作。
如果 `键` 不存在，将会创建一个新的键及对应的哈希表。
如果 `字段` 不存在则在操作进行前将其值设定成 `0` 。

 `HINCRBY` 的最大值限制为64位有符号整数。

@return

@integer-reply: 自增操作后 `字段` 的值。

@examples

如果指定了 `自增数` 参数，可以执行如下自增和自减操作：

```cli
HSET myhash field 5
HINCRBY myhash field 1
HINCRBY myhash field -1
HINCRBY myhash field -10
```

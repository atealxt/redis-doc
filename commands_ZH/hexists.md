返回 `字段` 是否存在于给定 `键` 的哈希表中。

@return

@integer-reply, 具体为：

* 如果哈希表包含 `字段` 返回 `1` 。
* 如果哈希表不包含 `字段` 或 `键` 不存在返回 `0` 。

@examples

```cli
HSET myhash field1 "foo"
HEXISTS myhash field1
HEXISTS myhash field2
```

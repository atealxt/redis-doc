将 `键` 重命名为 `新键` ，当且仅当 `新键` 不存在时。
当 `键` 不存在时返回错误。

**注意：** 在Redis 3.2.0 版本之前如果源键和目标键名称相同将返回错误。

@return

@integer-reply, 具体为：

* 如果 `键` 重命名为了 `新键` 返回 `1` 。
* 如果 `新键` 已经存在返回 `0` 。

@examples

```cli
SET mykey "Hello"
SET myotherkey "World"
RENAMENX mykey myotherkey
GET myotherkey
```

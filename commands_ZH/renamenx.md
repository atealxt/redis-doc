将 `键` 重命名为 `新键` ，当且仅当 `新键` 不存在时。
错误处理与 `RENAME` 相同。

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

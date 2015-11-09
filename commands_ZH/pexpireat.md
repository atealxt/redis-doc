`PEXPIREAT` 有着和 `EXPIREAT` 相同的效果和语义，除了将键的Unix时间过期单位从秒换成了毫秒。

@return

@integer-reply, 具体为：

* 如果设置了超时时间则返回 `1` 。
* 如果 `键` 不存在或无法设定超时时间（见： `EXPIRE` ）则返回 `0` 。

@examples

```cli
SET mykey "Hello"
PEXPIREAT mykey 1555555555005
TTL mykey
PTTL mykey
```

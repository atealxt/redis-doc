此命令的作用非常像 `EXPIRE` ，除了将键的生存时间单位从秒换成了毫秒。

@return

@integer-reply, 具体为：

* 如果设置了超时时间则返回 `1` 。
* 如果 `键` 不存在则返回 `0` 。

@examples

```cli
SET mykey "Hello"
PEXPIRE mykey 1500
TTL mykey
PTTL mykey
```

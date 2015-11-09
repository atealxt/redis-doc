删除 `键` 既有的超时设置，将键从 _不稳定的_ （有过期设置的键） 转变成 _持久的_ （键永不过期，没有关联的超时设置）。

@return

@integer-reply, 具体为：

* 如果移除了超时设置则返回 `1` 。
* 如果 `键` 不存在或没有超时设置则返回 `0` 。

@examples

```cli
SET mykey "Hello"
EXPIRE mykey 10
TTL mykey
PERSIST mykey
TTL mykey
```

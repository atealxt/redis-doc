设置 `键` 持有字符串 `值` 并设置 `键` 在给定的秒数后超时。
此命令等同于执行以下命令：

```
SET mykey value
EXPIRE mykey seconds
```

`SETEX` 是原子的，可以通过在 `MULTI` / `EXEC` 块内执行上面的两个命令再现。
它是比给出的序列操作更快速的可选方案，因为当把Redis作为缓存时这个操作非常通用。

当 `秒` 非法时返回错误。

@return

@simple-string-reply

@examples

```cli
SETEX mykey 10 "Hello"
TTL mykey
GET mykey
```

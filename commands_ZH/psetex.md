`PSETEX` 的作用非常像 `SETEX` ，唯一的不同就是将过期时间单位从秒换成了毫秒。

@examples

```cli
PSETEX mykey 1000 "Hello"
PTTL mykey
GET mykey
```

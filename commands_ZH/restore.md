创建一个键并关联一个由提供的序列化值（由 `DUMP` 得到）进行反序列化而获得的值。

如果 `ttl` 为0，键将不会设置过期时间，否则将被设置指定的过期时间（以毫秒为单位）。

如果 `key` 已经存在，除非使用 `REPLACE` 修改器，执行 `RESTORE` 会返回一个 "目标键忙碌" 错误。

`RESTORE` 会验证RDB的版本以及数据校验码。
如果不符合将返回错误。

@return

@simple-string-reply: 命令成功时返回OK。

@examples

```
redis> DEL mykey
0
redis> RESTORE mykey 0 "\n\x17\x17\x00\x00\x00\x12\x00\x00\x00\x03\x00\
                        x00\xc0\x01\x00\x04\xc0\x02\x00\x04\xc0\x03\x00\
                        xff\x04\x00u#<\xc0;.\xe9\xdd"
OK
redis> TYPE mykey
list
redis> LRANGE mykey 0 -1
1) "1"
2) "2"
3) "3"
```

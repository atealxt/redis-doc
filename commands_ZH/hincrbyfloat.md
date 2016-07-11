让存储在 `键` 的哈希表中 `字段` 的数进行指定浮点数 `自增数` 的自增操作。
如果自增值为负数，将进行 **自减** 而不是自增。
如果字段不存在则在操作进行前将其值设定成 `0` 。
发生以下情形将返回错误：

* 字段的值类型错误（不是字符串）。
* 字段的值或自增时无法将其转化为双精度浮点数。

此命令的行为与 `INCRBYFLOAT` 相同，更多的信息请参照 `INCRBYFLOAT` 的文档。

@return

@bulk-string-reply: 自增操作后 `字段` 的值。

@examples

```cli
HSET mykey field 10.50
HINCRBYFLOAT mykey field 0.1
HINCRBYFLOAT mykey field -5
HSET mykey field 5.0e3
HINCRBYFLOAT mykey field 2.0e2
```

## 实现细节

此命令在复制链接及只可追加文件中进行传播时总是使用 `HSET` 操作，所以不会因基础浮点运算而带来不一致性。

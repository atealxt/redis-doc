合并多个HyperLogLog的值并估算集合的势。

将合并后的HyperLogLog存入目标键，如果键不存在则会创建（并设一个空的HyperLogLog）。

@return

@simple-string-reply: 此命令仅会返回 `OK` 。

@examples

```cli
PFADD hll1 foo bar zap a
PFADD hll2 a b c foo
PFMERGE hll3 hll1 hll2
PFCOUNT hll3
```

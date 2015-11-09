返回指定Redis集群哈希槽的键的数量。
此命令只查询本地数据集，所以访问没有处于该哈希槽的节点的返回值为零。

```
> CLUSTER COUNTKEYSINSLOT 7000
(integer) 50341
```

@return

@integer-reply: 返回指定哈希槽的键的数量，或在哈希槽无效时返回错误。

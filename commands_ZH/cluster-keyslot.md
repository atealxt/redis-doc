返回指定键的对应哈希槽整数值。
本API暴露了Redis底层的哈希算法实现，主要用于调试和测试。
命令的用例：

1. 客户端库可以使用Redis测试它自己的哈希算法，生成随机的键并使用本地实现，和使用命令 `CLUSTER KEYSLOT` 比较结果是否一样。
2. 用户可以使用此命令查看指定键的哈希槽是什么，关联的Redis集群节点是哪个。

## 例子

```
> CLUSTER KEYSLOT somekey
11058
> CLUSTER KEYSLOT foo{hash_tag}
(integer) 2515
> CLUSTER KEYSLOT bar{hash_tag}
(integer) 2515
```

注意此命令完整的实现了哈希算法，包括对Redis集群键哈希算法的特别属性 **哈希标签** 的支持。如果在键名中包含模式，就可以只对模式即在 `{` 与 `}` 之间的部分进行散列，这样一来同一个实例就可以处理多个键。

@return

@integer-reply: 哈希槽编号。

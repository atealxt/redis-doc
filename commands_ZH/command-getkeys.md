从一条完整的Redis命令中返回所有键 @array-reply 。

`COMMAND GETKEYS` 是一个辅助命令用来从完整的Redis命令中寻找其中的键。

`COMMAND` 显示的一些命令的键位置不确定，意味着该命令必须进行解析以操作存储或检索键。
可以直接使用 `COMMAND GETKEYS` 发现键的所在位置，理解Redis是如何解析命令的。


@return

@array-reply: 给定命令的键列表。

@examples

```cli
COMMAND GETKEYS MSET a b c d e f
COMMAND GETKEYS EVAL "not consulted" 3 key1 key2 key3 arg1 arg2 arg3 argN
COMMAND GETKEYS SORT mylist ALPHA STORE outlist
```

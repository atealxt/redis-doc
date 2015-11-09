返回多个Redis命令的内容 @array-reply 。

返回结果的格式与 `COMMAND` 相同并且可以指定返回哪些命令。

如果查询不存在的命令，返回nil。


@return

@array-reply: 命令内容的嵌套列表。

@examples

```cli
COMMAND INFO get set eval
COMMAND INFO foo evalsha config bar
```

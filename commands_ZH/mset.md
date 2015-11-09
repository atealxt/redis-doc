对给定的键分别设置相应的值。
`MSET` 将用新值替换已经存在的值，就像 `SET` 的规则一样。
如果不想重写既有的值请参见 `MSETNX` 。

`MSET` 为原子性，所以给定的所有键都将同时进行设置。
客户端不可能遇到有些键被更新了而其他键没有改变的情况。

@return

@simple-string-reply: 总是 `OK` ， `MSET` 不会执行失败。

@examples

```cli
MSET key1 "Hello" key2 "World"
GET key1
GET key2
```

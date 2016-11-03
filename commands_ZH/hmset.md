为存储在 `键` 中的哈希表内的指定字段设值。
此命令对哈希表中已存在的指定字段进行重写。
如果 `键` 不存在，将会创建一个新的持有哈希表的键。

@return

@simple-string-reply

@examples

```cli
HMSET myhash field1 "Hello" field2 "World"
HGET myhash field1
HGET myhash field2
```

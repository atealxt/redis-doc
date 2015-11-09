返回存储在 `键` 的 `字段` 关联值的字符串长度。 如果 `键` 或 `字段` 不存在，返回0。 

@return

@integer-reply: `字段` 关联值的字符串长度，或当哈希表中没有 `字段` 或 `键` 不存在时返回零。

@examples

```cli
HMSET myhash f1 HelloWorld f2 99 f3 -256
HSTRLEN myhash f1
HSTRLEN myhash f2
HSTRLEN myhash f3
```

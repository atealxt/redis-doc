返回 `成员` 是否是存储在 `键` 的集合的成员。

@return

@integer-reply, 具体为：

* `1` 如果元素是集合的成员。
* `0` 如果元素不是集合的成员，或 `键` 不存在。

@examples

```cli
SADD myset "one"
SISMEMBER myset "one"
SISMEMBER myset "two"
```

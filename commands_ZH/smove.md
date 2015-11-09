将 `成员` 从 `源` 集合移动到 `目标` 集合。
此操作为原子性的。
任何时候元素将作为成员出现在 `源` 集合 **或** 其他客户端的 `目标` 集合中。

如果源集合不存在或不包含指定元素，不执行任何操作并返回 `0` 。
否则，元素将从源集合中删除并添加到目标集合。
当指定元素已经存在于目标集合时，只将其从源集合中删除。

如果 `源` 或 `目标` 的值类型不是集合时返回错误。

@return

@integer-reply, 具体为：

* `1` 如果移动了元素。
* `0` 如果元素不是 `源` 的成员，不执行任何操作。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myotherset "three"
SMOVE myset myotherset "two"
SMEMBERS myset
SMEMBERS myotherset
```

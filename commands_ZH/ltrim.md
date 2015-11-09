对既有列表进行截取以便只含有特定区间的元素。
  `起始` 和 `结束` 是以零开始的索引， `0` 意味着列表的第一个元素（列表首元素）， `1` 意味着列表的下一个元素，以此类推。

例如： `LTRIM foobar 0 2` 将会修改存储在 `foobar` 的列表，只保留列表的前两个元素。

`起始` 和 `结束` 也可以是负数代表从列表尾部进行索引，即 `-1` 是列表的最后一个元素， `-2` 是倒数第二个元素，以此类推。

超出范围的索引将不会产生错误：如果 `起始` 索引大于列表末端的位置，或  `起始 > 结束` ，将返回空列表（将导致 `键` 被删除）。
如果 `结束` 索引大于列表末端的位置，Redis将当成列表末节点索引进行处理。

 `LTRIM` 的一种常用用法是结合 `LPUSH` / `RPUSH` 来使用。
例如：

```
LPUSH mylist someelement
LTRIM mylist 0 99
```

这对命令将向列表中增加一个新元素，并确保列表的元素数量不超过100个。
这将会很有用比如使用Redis存储日志。
值得注意的一点是，这样使用 `LTRIM` 是一个 O(1) 操作，因为平均上只会从列表尾部删除一个元素。

@return

@simple-string-reply

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LTRIM mylist 1 -1
LRANGE mylist 0 -1
```

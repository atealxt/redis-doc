返回存储在 `键` 的列表中指定区间的元素。
偏移量 `起始` 和 `结束` 是以零开始的索引， `0` 意味着列表的第一个元素（列表首元素）， `1` 意味着列表的下一个元素，以此类推。

偏移量也可以使用负数来代表从列表末尾进行偏移。
例如， `-1` 是列表的最后一个元素， `-2` 是倒数第二个元素，以此类推。

## 在不同编程语言中的范围函数一致性

注意如果有一个0到100的数字列表， `LRANGE list 0 10` 将会返回11个元素，就是说，包含最右面的那个元素。
这 **或许** 与你选择的编程语言中范围相关函数的行为是一致的（请考虑Ruby的 `Range.new` 、 `Array#slice` 函数，或Python的 `range()` 函数）。

## 超出范围的索引

超出范围的索引将不会产生错误。
如果 `起始` 索引大于列表末端的位置，将返回空列表。
如果 `结束` 索引大于列表末端的位置，Redis将当成列表末节点索引进行处理。

@return

@array-reply: 指定范围的列表元素。

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LRANGE mylist 0 0
LRANGE mylist -3 2
LRANGE mylist -100 100
LRANGE mylist 5 10
```

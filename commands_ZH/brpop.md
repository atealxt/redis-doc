`BRPOP` 为阻塞的列表弹出命令。
它是阻塞版本的 `RPOP` ，当给定的列表中没有元素可弹出时将它会阻塞住连接。
从给定键的顺序开始检查，从第一个非空列表的队尾弹出一个元素。

准确的语义请参见[BLPOP 文档][cb]， `BRPOP` 和 `BLPOP` 相同，唯一的区别是从列表的尾部而不是首部弹出元素。

[cb]: /commands/blpop

@return

@array-reply: 具体为：

* 当没有元素可被弹出且超时时间过期时返回 `nil` 数组。
* 返回由两个元素组成的数组。第一个元素是弹出值所在列表的键，第二个元素是弹出值。

@examples

```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BRPOP list1 list2 0
1) "list1"
2) "c"
```

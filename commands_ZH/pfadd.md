保存指定的元素至HyperLogLog，第一个参数为指向该结构体的键，其余为添加元素。

此命令可能会触发HyperLogLog内部重新估算元素数量（集合的势）。

如果命令执行后估算的结果变化了， `PFADD` 返回1，否则返回0。
如果指定的键不存在，命令将会自动创建一个空的HyperLogLog结构（一个指定长度和编码的Redis字符串）。

调用命令时只传键名不传元素也是合法的，如果键存在则不会执行任何操作，否则只创建数据结构（并返回1）。

关于HyperLogLog数据结构的介绍请见 `PFCOUNT` 命令。

@return

@integer-reply, 具体为：

* 如果HyperLogLog内部至少有一条记录改变了的话返回1。否则返回0。

@examples

```cli
PFADD hll a b c d e f g
PFCOUNT hll
```

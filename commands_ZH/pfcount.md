当使用单个键调用时返回存储在键的HyperLogLog的势，当键不存在时返回0。

当使用多个键调用时，内部将多个键所在的HyperLogLog合并成一个临时HyperLogLog并返回其势。

HyperLogLog数据结构可以用于在一个集合中计算 **唯一** 元素的个数并只使用少量常数大小的内存，每个HyperLogLog占用12k字节（加上少量字节用于键本身）。

返回的势不是精确的，一般错误率在0.81%。

例如为了取得每天的独立搜索请求的数量， 需要执行一个程序在每次搜索时调用 `PFADD` 。
那么可以在任何时候使用 `PFCOUNT` 取得估算出的独立请求数。

注意：此方法的副作用是，HyperLogLog的最后8个字节作为上次计算的势的缓存，调用此命令可能会修改HyperLogLog。

@return

@integer-reply, 具体为：

* 使用 `PFADD` 添加的独立元素估算数量。

@examples

```cli
PFADD hll foo bar zap
PFADD hll zap zap zap
PFADD hll foo bar
PFCOUNT hll
PFADD some-other-hll 1 2 3
PFCOUNT hll some-other-hll
```

性能
---

当使用单个键调用 `PFCOUNT` 时，性能非常高，即使理论中说了处理密集HyperLogLog消耗的常数时间很长。
这是因为 `PFCOUNT` 使用缓存保存了上一次计算的势，它很少改变，大多数 `PFADD` 操作都不会更新任何记录。
每秒上百次操作是可以实现的。

当使用多个键调用 `PFCOUNT` 时，将即时进行HyperLogLog合并，速度很慢并且结果势无法缓存。
使用多个键调用 `PFCOUNT` 可能会花费毫秒数量级的时间，不能滥用。

用户应该记住单键执行和多键执行是两种不同的用法，性能也是不一样的。

HyperLogLog的结构
---

Redis HyperLogLog有两种实现：
 *稀疏* 结构适用于记录少量元素（少量非零值的数据）， *密集* 结构适用于大量元素。
需要时Redis会自动从稀疏结构切换到密集结构。

稀疏结构使用行程长度编码进行优化，在存储大量数据为零的时候非常高效。 
密集结构使用12288个字节的Redis字符串存储16384个6比特的计数器。
使用两种结构是因为如果用12k（密集结构需要的内存数量）对较小势的少量数据进行编码是很低效的。

两种结构都有一个16字节的前缀，包含一个魔数、一个表示编码和版本的字段、还有缓存的计算结果势，以小端法格式进行存储（在HyperLogLog更新后计算的结果无效时比特值仅为一个1）。

作为一个Redis的字符串，HyperLogLog可以使用 `GET` 获取并用 `SET` 进行设置。
向错误的HyperLogLog调用 `PFADD` 、 `PFCOUNT` 或 `PFMERGE` 也没有关系，返回值可能是个随机数，不会影响服务器的稳定性。
当错误使用稀疏结构时，大多数情况下服务器会正确识别并返回错误。

HyperLogLog无关于处理器的字长及字节顺序，32位和64位处理器的HyperLogLog结构相同，不论是高位在前还是低位在前。

更多关于Redis HyperLogLog实现的内容请见 [此博文](http://antirez.com/news/75)。
 `hyperloglog.c` 是实现的源码，易于阅读和理解，包括对稀疏结构及密集结构的完整解释。

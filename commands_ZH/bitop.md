对多个键（值为字符串）执行按位操作，并把结果保存至目标键。

 `BITOP` 命令支持四种按位操作：**AND**，**OR**，**XOR**和**NOT**，合法的调用方式为：

* `BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN`
* `BITOP OR  destkey srckey1 srckey2 srckey3 ... srckeyN`
* `BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN`
* `BITOP NOT destkey srckey`

正如所见 **NOT** 比较特别，它只接受一个输入键，因为它对比特取反所以只能是一元操作。

操作的结果总是会保存至 `destkey` 。

## 对不同长度字符串的处理

当操作不同长度的字符串时，所有比最长的那个字符串短的字符串都将以零值补齐长度后再进行处理。

不存在的键也将进行相同处理，它会被当作一个以零值填充的与最长字符串等长的字符串。

@return

@integer-reply

存储在目标键内字符串的长度，即最长输入字符串的长度。

@examples

```cli
SET key1 "foobar"
SET key2 "abcdef"
BITOP AND dest key1 key2
GET dest
```

## 模式：使用位图进行实时统计

`BITOP` 是一个在 `BITCOUNT` 命令文档中描述的模式的很好补充。
多个位图可以进行合并以获得一个总的计数结果。

请见一篇描述了一个有趣用例的文章：“[使用Redis位图结构进行快速而易用的实时统计][hbgc212fermurb]”。

[hbgc212fermurb]: http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps

## 性能方面的考虑

`BITOP` 是一个可能会很慢的命令，它的运行时间复杂度为O(N)。
在进行长输入字符串运算时要小心。

在实时度量与统计中使用长字符串输入时可以考虑把运算放到从服务器上（并禁用其只读选项），这样一来就可以避免按位运算阻塞主服务器实例。

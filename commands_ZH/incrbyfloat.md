让储存在 `键` 中表示浮点数的字符串递增 `自增数` 指定的值。
如果 `increment` 值为负数，值结果将自减。
如果键不存在，在执行操作前将其置为 `0` 。
发生以下情形将返回错误：

* 键容纳的值类型错误（不是字符串）。
* 键的值或指定递增值无法转换为双精度浮点数。

如果命令成功，递增后的值将作为新值存储在键上（替代旧的），并以字符串的形式返回给调用者。

字符串键容纳的值和递增参数可以用指数计数法的方式提供，然而在递增计算之后值会一致的以相同格式进行存储，即，一个整数值后跟着一个点（如果需要的话），和一个可变数目的数字代表小数部分。
尾部的零将被移除。

不论实际内部计算的精度如何，输出的精度固定为小数点后17位。

@return

@bulk-string-reply: `键` 自增后的值。

@examples

```cli
SET mykey 10.50
INCRBYFLOAT mykey 0.1
INCRBYFLOAT mykey -5
SET mykey 5.0e3
INCRBYFLOAT mykey 2.0e2
```

## 实现细节

此命令在复制链接及只可追加文件中进行传播时总是使用 `SET` 操作，所以不会因基础浮点运算而带来不一致性。

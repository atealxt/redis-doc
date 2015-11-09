**警告**：此命令被重命名为了 `GETRANGE` ， `<= 2.0` 的版本此命令叫做 `SUBSTR` 。

返回存储在 `键` 的字符串区间值，值取决于 `起始` 与 `结束` 偏移量（包含二者）。
可以使用负值偏移量，来提供从字符串末尾开始计算的偏移量。
即-1代表最后一个字符，-2代表倒数第二个，以此类推。

此功能对于超出范围的请求，将把结果取值范围限制在字符串的真实长度之内。

@return

@bulk-string-reply

@examples

```cli
SET mykey "This is a string"
GETRANGE mykey 0 3
GETRANGE mykey -3 -1
GETRANGE mykey 0 -1
GETRANGE mykey 10 100
```

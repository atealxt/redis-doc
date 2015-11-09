如果 `键` 已经存在且值为字符串，此命令将把 `值` 追加到字符串的末尾。
如果 `键` 不存在会创建并设一个空字符串，这样一来 `APPEND` 类似于 `SET` 命令。

@return

@integer-reply: 执行追加操作后字符串的长度。

@examples

```cli
EXISTS mykey
APPEND mykey "Hello"
APPEND mykey " World"
GET mykey
```

## 模式: 时间序列

 `APPEND` 命令可以用非常紧凑的方式处理定长数据，常见于 _时间序列_ 。
 每当有新值出现时可以使用以下命令进行存储：

```
APPEND timeseries "fixed-size sample"
```

在时间序列中访问单独的元素并不难：

* `STRLEN` 可以用来获得定长数据的个数。
* `GETRANGE` 可以用来访问任意元素。
  如果时间序列里包含相互关联信息的话，就可以结合 `GETRANGE` 和Redis 2.6版本可用的Lua脚本引擎很容易的实现二分查找。
* `SETRANGE` 可以用来重写既有的时间序列。

本模式的限制是只能使用追加操作，而没有办法轻松的将时间序列裁剪为指定的大小，因为Redis目前缺少修剪字符串对象的命令。
但是，时间序列对空间的利用率则是非常高的。

提示：可以基于当前Unix时间生成不同的键，这样可以只使用平均相对较少的键，来避免维护大量的键。让本模式对分布式多Redis实例更加友好。

以下是一个温度传感器使用固定长度字符串进行抽样的例子（在真实环境中使用二进制格式更好）。

```cli
APPEND ts "0043"
APPEND ts "0035"
GETRANGE ts 0 3
GETRANGE ts 4 7
```

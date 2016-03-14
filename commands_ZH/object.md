 `OBJECT` 命令允许查看键所对应Redis对象的内部情况。
这有助于调试，或了解键是否使用了特殊编码的数据类型来节省空间。
当把Redis用作缓存时，应用程序也可以使用 `OBJECT` 命令返回的报告信息来实现应用级别的键淘汰策略。

 `OBJECT` 命令支持多种子命令：

* `OBJECT REFCOUNT <key>` 返回给定键的值的引用数。
  此命令主要用于调试。
* `OBJECT ENCODING <key>` 返回键对应存储值的内部表示。
* `OBJECT IDLETIME <key>` 返回给定键的值自存储后闲置的秒数（没有读或写的请求操作）
  虽然返回值以秒为单位但实际计时器是以10秒为单位，并且可能在未来的实现中改变。

对象可以以不同的方式进行编码：

* 字符串可以编码为 `raw` （普通字符串格式）或 `int` （为了节省空间而编码的代表64位有符号整数的字符串）。
* 列表可以编码为 `ziplist` 或 `linkedlist` 。
   `ziplist` 是一种用来为较小列表节省空间的特殊编码。
* 集合可以编码为 `intset` 或 `hashtable` 。
   `intset` 是一种为仅以整数组成的较小集合而设的特殊编码。
* 散列表可以编码为 `ziplist` 或 `hashtable` 。
   `ziplist` 是一为较小散列表而设的特殊编码。
* 有序集合可以编码为 `ziplist` 或 `skiplist` 。
  较小的有序集合可以像列表一样使用 `ziplist` 这种特殊编码，而 `skiplist` 编码则可用于任何大小的有序集合。

当执行的操作使得Redis无法继续使用节省空间的编码时，所有特殊的编码类型将会自动转换成普通类型。

@return

不同的子命令返回不同的返回值。

* 子命令 `refcount` 和 `idletime` 返回整数。
* 子命令 `encoding` 返回bulk应答。

如果尝试检查的对象不存在，返回null bulk应答。

@examples

```
redis> lpush mylist "Hello World"
(integer) 4
redis> object refcount mylist
(integer) 1
redis> object encoding mylist
"ziplist"
redis> object idletime mylist
(integer) 10
```

从下面的例子中可以看出，当Redis不能继续使用节省空间的编码时是如何改变编码的。

```
redis> set foo 1000
OK
redis> object encoding foo
"int"
redis> append foo bar
(integer) 7
redis> get foo
"1000bar"
redis> object encoding foo
"raw"
```

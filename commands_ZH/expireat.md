`EXPIREAT` 与 `EXPIRE` 有着相同的作用与语义，除了以绝对 [Unix时间戳][hewowu]（自1970年1月1日以来经过的秒数）取代指定TTL（生存时间）秒数。

[hewowu]: http://en.wikipedia.org/wiki/Unix_time

关于命令的详细解释请参见 `EXPIRE` 文档。

## 背景

引入 `EXPIREAT` 是为了AOF的持久化模式，将相对的超时时间转换为绝对时间。
当然，它也可以直接用在为键设置一个指定的在将来超时的时间。

@return

@integer-reply, 具体为：

* 如果超时设置成功返回 `1` 。
* 如果键不存在或无法设置超时时间返回 `0` （请见 `EXPIRE` ）。

@examples

```cli
SET mykey "Hello"
EXISTS mykey
EXPIREAT mykey 1293840000
EXISTS mykey
```

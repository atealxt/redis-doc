返回具有超时设置的键的剩余生存时间。
此自省技能可以让Redis客户端确认给定的键还能有多长时间存在于数据集中。

如果键不存在或没有设置过期时间，在Redis 2.6或更早版本中返回 `-1` 。

从Redis 2.8开始当发生错误时返回值变为：

* 如果键不存在命令返回 `-2` 。
* 如果键存在但没有设置过期时间返回 `-1` 。

另外也可以使用 `PTTL` 命令，不同的是它是以毫秒为单位（只在Redis 2.6及以上版本中可用）。

@return

@integer-reply: 返回剩余的生存时间秒数，或返回负数表示错误（键上面的描述）。

@examples

```cli
SET mykey "Hello"
EXPIRE mykey 10
TTL mykey
```

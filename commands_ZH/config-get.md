`CONFIG GET` 命令用来读取处于运行中的Redis服务的配置参数。
Redis 2.4并不支持读取所有的配置参数，但在Redis 2.6可以通过使用本命令读取全部的配置信息。

与此配套的的命令 `CONFIG SET` 可以用来在运行时修改配置。

`CONFIG GET` 接受一个glob风格的模式参数。
所有匹配的配置参数会封装为一个键-值对列表并返回。
例如：

```
redis> config get *max-*-entries*
1) "hash-max-zipmap-entries"
2) "512"
3) "list-max-ziplist-entries"
4) "512"
5) "set-max-intset-entries"
6) "512"
```

你可以通过在一个打开的 `redis-cli` 提示符下输入 `CONFIG GET *` 来获得一个所有支持的配置参数列表。

所有支持的参数与在 [redis.conf][hgcarr22rc] 文件中使用的配置参数具有等价的含义，以下是几个重要的不同点：

[hgcarr22rc]: http://github.com/antirez/redis/raw/2.8/redis.conf

* 字节数或其他数量无法使用 `redis.conf` 里的缩写格式（`10k` ， `2gb` ... 等等），每个数都要指定成为具有良好格式的64-位整型值，即配置指令中的基本单位。
* 保存参数是一个由空格分隔的整数字符串。
  每对整数代表一个 秒数/修改数 的阀值。

例如 `redis.conf` 中有如下配置：

```
save 900 1
save 300 10
```

它的意思是，如果至少有1个数据在900秒间修改了的话那么时间过后会进行保存操作，如果至少有10个数据在300秒间修改了的话时间过后会进行保存操作，使用 `CONFIG GET` 命令将会返回"900 1 300 10"。

@return

命令的返回类型为@array-reply。

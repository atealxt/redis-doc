返回上一次成功保存DB的UNIX时间。
客户端可以读取 `LASTSAVE` 的值来检查 `BGSAVE` 命令是否执行成功，通过发出一个 `BGSAVE` 命令并周期性的每N秒检查 `LASTSAVE` 的值是否改变。

@return

@integer-reply: UNIX时间戳。

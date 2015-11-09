Redis配置
===

Redis启动可以不指定配置文件而使用内置默认配置，但仅推荐用于开发及测试。

正确配置Redis的方法是提供一个Redis配置文件，一般为 `redis.conf` 。

文件 `redis.conf` 包含一系列指令，格式非常简单：

    keyword argument1 argument2 ... argumentN

一个配置指令的例子：

    slaveof 127.0.0.1 6380

参数字符串包含空格时可以使用双引号，见下面的例子：

    requirepass "hello world"

配置指令的列表，以及它们的意义和用法请见例子文件redis.conf，其已包含了Redis分布式方面的信息。

* 自文档 [Redis 3.0的redis.conf](https://raw.githubusercontent.com/antirez/redis/3.0/redis.conf)
* 自文档 [Redis 2.8的redis.conf](https://raw.githubusercontent.com/antirez/redis/2.8/redis.conf)
* 自文档 [Redis 2.6的redis.conf](https://raw.githubusercontent.com/antirez/redis/2.6/redis.conf)
* 自文档 [Redis 2.4的redis.conf](https://raw.githubusercontent.com/antirez/redis/2.4/redis.conf)

通过命令行传递参数
---

Redis从2.6开始也可以直接使用命令行传递Redis配置参数。
这在测试时非常有用。
以下是启动Redis新实例，并使用端口6380作为运行在127.0.0.1、端口6379上实例的从服务的例子。

    ./redis-server --port 6380 --slaveof 127.0.0.1 6379

通过命令行传递的参数格式与在redis.conf文件中使用的格式完全相同，只是需要给每个关键词加上 `--` 前缀。

参数在被转换成redis.conf文件的格式后，系统会在内存中临时生成一个配置文件（可能链接着用户传递的配置文件）。

在服务运行时修改Redis配置
---

在运行时可以修改配置而不用停止或重启服务，也可以使用专门的命令 [CONFIG SET](/commands/config-set) 及 [CONFIG GET](/commands/config-get) 来灵活的查询当前配置。

不是所有的配置指令都支持运行时修改，但大部分都还是支持的。
更多信息请见 [CONFIG SET](/commands/config-set) 及 [CONFIG GET](/commands/config-get) 页面。

注意在运行时修改的配置 **不会作用于redis.conf文件** ，所以当下一次重启时Redis将会重新使用旧的配置。

请确认在使用 [CONFIG SET](/commands/config-set) 后，同时修改 `redis.conf` 文件中的相应配置。
可以手动执行，从Redis 2.8也可以使用 [CONFIG REWRITE](/commands/config-rewrite) 自动扫描文件 `redis.conf` 更新与当前配置不同的字段。
配置中不存在但值是默认值的字段不会被添加进去。原配置文件中的注释将会保留。

把Redis配置成缓存系统
---

如果计划仅把Redis当成缓存系统使用，每个键都有过期设置，可以考虑使用下面的配置（例子中假设最大内存限制为2兆）：

    maxmemory 2mb
    maxmemory-policy allkeys-lru

在此配置下不需要应用程序使用 `EXPIRE` 命令（或equivalent）为键设置生存时间，当达到2兆内存限制时，系统使用近似LRU算法清除所有需要清除的键。

基本上来说在这种配置下Redis的行为类似于memcached。
这有一篇更深入介绍缓存的文章 [使用Redis作为LRU缓存] (/topics/lru-cache) 。

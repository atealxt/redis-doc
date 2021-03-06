`CONFIG REWRITE` 命令用来重写服务启动时使用的 `redis.conf` 文件，通过对配置文件进行最小化改动以和正在运行服务保持一致。
由于 `CONFIG SET` 命令的使用，生成的文件可能和原来的不一样。

重写使用了非常谨慎的方法进行：

* 原始 redis.conf 的注释及整体结构都将尽可能的进行保留。
* 如果选项已经存在于旧的 redis.conf 文件中，将会在相同位置进行重写（行号）。
* 如果选项还没有出现过，但是值设为了默认值，将不会被添加到重写方法中。
* 如果选项还没有出现过，也不是默认值，将会被追加到文件的末尾。
* 没有效果的行将会被擦除。例如文件中有多个 `save` 相关的指令，但当前运行的配置中没有或只有很少关于它的设置（比如关闭了RDB持久化），所有的相关行都会被擦除。

即使原始配置文件由于某些原因丢失了，CONFIG REWRITE命令也可以依据现有状态重新生成一个配置文件。
但如果在服务器启动时根本没有使用任何配置文件，执行CONFIG REWRITE就会返回错误。

## 原子性重写操作

为了确保 redis.conf 文件的一致性，在发生错误或宕机时文件不会被重写。
只有当即将生成的文件内容大于等于旧的文件时，执行一个 `write(2)` 后旧文件才会被替换。
有时为了确保结果文件足够大，会在末尾额外添加一些注释，在新旧文件替换时再删除掉。

@return

@simple-string-reply: 当配置被正确重写后返回 `OK` 。否则返回错误。

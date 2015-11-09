在后台保存数据库。
立即返回OK状态码。
Redis创建分支，父进程持续为客户端提供服务，子进程保存数据库至磁盘并在完成时退出。
客户端可以使用 `LASTSAVE` 命令来验证操作是否成功。

详细信息请参见[持久化文档][tp]。

[tp]: /topics/persistence

@return

@simple-string-reply

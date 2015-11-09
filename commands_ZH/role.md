显示Redis实例在复制中的角色，返回值为 `master` 、 `slave` 或 `sentinel` 。
同时命令返回关于复制的状态信息（如果角色是master或slave），或正在监控的主服务名字列表（如果角色是sentinel）。

## 输出格式

命令返回一个元素数组。第一个元素是实例的角色，值为一下三个的其中一个：

* "master"
* "slave"
* "sentinel"

数组中的其他元素根据角色的不同各不相同。

## Master的输出

一个在master实例中调用 `ROLE` 的例子：

```
1) "master"
2) (integer) 3129659
3) 1) 1) "127.0.0.1"
      2) "9001"
      3) "3129242"
   2) 1) "127.0.0.1"
      2) "9002"
      3) "3129543"
```

master输出由以下几部分组成：

1. 字符串 `master` 。
2. 当前主服务的复制偏移量，用于主从服务的部分重同步交互，从服务根据此偏移量继续进行复制。
3. 由三个元素组成的代表从服务连接的数组。每个子数组的三个元素包含从服务的IP、端口及最后一次复制后的偏移量。

## Slave的输出

一个在slave实例中调用 `ROLE` 的例子：

```
1) "slave"
2) "127.0.0.1"
3) (integer) 9000
4) "connected"
5) (integer) 3167038
```

slave输出由以下几部分组成：

1. 字符串 `slave` 。
2. 主服务的IP。
3. 主服务的端口。
4. 主服务端报告的复制状态，值为 `connect` （需要和主服务进行连接）、 `connecting` （正在连接主服务）、 `sync` （正在进行同步）或 `connected` （已同步）。
5. 根据主服务复制的偏移量计算得出的到目前为止的数据接收量。

## Sentinel的输出

一个Sentinel输出的例子：

```
1) "sentinel"
2) 1) "resque-master"
   2) "html-fragments-master"
   3) "stats-master"
   4) "metadata-master"
```

sentinel输出由以下几部分组成：

1. 字符串 `sentinel` 。
2. 当前Sentinel实例正在监控的主服务的数组。

@return

@array-reply: 第一个元素为 `master` 、 `slave` 或 `sentinel` ，剩余元素根据角色的不同值也不同，在上面的描述中已说明。

@history

* 此命令在Redis稳定版的中间版本2.8.12中引入。

@examples

```cli
ROLE
```

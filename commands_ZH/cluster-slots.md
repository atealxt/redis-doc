`CLUSTER SLOTS` 返回关于Redis集群实例映射关系的详细信息。
本命令适合在Redis集群客户端库中使用，取得（或在接收到重定向时更新）集群 *哈希槽* 与实际网络节点地址（由IP地址及TCP端口组成）的映射关系。
这样当接收到一个命令后，可以依据键的情况把命令发送至某个实例。

## 嵌套的结果集数组
每个嵌套的结果为：

  - 槽的起始位置
  - 槽的结束位置
  - 槽的主实例，以嵌套的IP/端口数组表示 
  - 主实例的第一个复制节点的槽范围
  - 第二个
  - ...直至所有复制节点。

每条结果包含主实例所有有效的复制节点的槽范围列表。
不包括失败的复制节点。

第三个嵌套返回结果为槽的主实例的IP/端口。在它之后的所有IP/端口均为复制节点的。

如果集群中有实例出现不连续的槽（例如1-400、900、1800-6000）那么主从将会对槽范围的顶层级进行冗余。

@return

@array-reply: 槽范围以及IP/端口映射的嵌套列表。

### 输出示例
```
127.0.0.1:7001> cluster slots
1) 1) (integer) 0
   2) (integer) 4095
   3) 1) "127.0.0.1"
      2) (integer) 7000
   4) 1) "127.0.0.1"
      2) (integer) 7004
2) 1) (integer) 12288
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7003
   4) 1) "127.0.0.1"
      2) (integer) 7007
3) 1) (integer) 4096
   2) (integer) 8191
   3) 1) "127.0.0.1"
      2) (integer) 7001
   4) 1) "127.0.0.1"
      2) (integer) 7005
4) 1) (integer) 8192
   2) (integer) 12287
   3) 1) "127.0.0.1"
      2) (integer) 7002
   4) 1) "127.0.0.1"
      2) (integer) 7006
```



`CLUSTER INFO` 提供了和 `INFO` 样式相同的Redis集群最重要参数的信息。
下面是一个样本输出，再下面是每个字段的描述。

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:2
cluster_stats_messages_sent:1483972
cluster_stats_messages_received:1483968
```

* `cluster_state`: 集群的状态。如果节点可以接收请求则为 `ok` ，如果有至少一个哈希槽未绑定（没有节点与其关联）、处于错误状态（节点被标记为FAIL）、或节点无法访问过半数的主服务，则为 `fail` 。
* `cluster_slots_assigned`: 已关联节点（不是未绑定的）的哈希槽数量。此数为16384时表示节点工作正常，也就是每个哈希槽都映射了节点。
* `cluster_slots_ok`: 节点映射状态不是 `FAIL` 或 `PFAIL` 的哈希槽数量。
* `cluster_slots_pfail`: 节点映射状态是 `PFAIL` 的哈希槽数量。注意这些哈希槽仍然是正常工作的，只要 `PFAIL` 状态不被失败检测算法提升为 `FAIL` 。 `PFAIL` 的意思只是当前无法联系到该节点，可能只是个暂时性错误。
* `cluster_slots_fail`: 节点映射状态是 `FAIL` 的哈希槽数量。如果不为零则节点无法提供服务除非配置项 `cluster-require-full-coverage` 为 `no` 。
* `cluster_known_nodes`: 集群的所有已知节点数量，包括状态为 `HANDSHAKE` 的有可能还未真正加入集群的节点。
* `cluster_size`: 集群中至少服务了一个哈希槽的主服务节点数量。
* `cluster_current_epoch`: 本地 `Current Epoch` 变量，用于在故障转移时创建递增的唯一版本号。
* `cluster_my_epoch`: 节点的 `Config Epoch` ，当前配置的版本号。
* `cluster_stats_messages_sent`: 通过集群的端-到-端二进制总线发送的消息数。
* `cluster_stats_messages_received`: 通过集群的端-到-端二进制总线接收的消息数。

更多关于当前Epoch和Epoch变量配置的信息请见Redis集群说明文档。

@return

@bulk-string-reply: 以两个字节 `CRLF` 分隔的多行 `<字段名>:<值>` 格式的字段名称与值的映射。 

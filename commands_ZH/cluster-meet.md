`CLUSTER MEET` 用于将支持Redis集群的节点连接至集群中。

命令的想法源于节点之间在初始情况下不互相信任，彼此未知，所以不太可能发生因系统管理错误或网络地址改变而导致不同集群之间的节点通信。

为了在指定节点之间建立联系组成Redis集群，只有这两个方法：

1. 系统管理员执行 `CLUSTER MEET` 命令主动让一个节点与另一个进行连接。
2. 已知节点在发送的gossip部分中包含未确认节点列表。如果接收节点信任并确认发送节点，就将处理gossip部分，与那些未知节点进行握手。

注意，Redis集群需要建立一张完整的网（每个节点与所有其他节点进行连接），但建立集群却不需要向所有节点发送 `CLUSTER MEET` 命令分别建立完整的关系网，而是只发送适量的 `CLUSTER MEET` ，通过 *已知节点链* 就能让每个节点互相可达。
感谢心跳包的gossip信息交换，丢失的链接会再次被创建。

那么，如果节点A与节点B通过 `CLUSTER MEET` 连接，然后B再与C连接，那么A与C会通过某一途径握手建立连接。

另一个例子：假设欲建立有四个节点A、B、C、D的集群，可以只向A发送如下命令：

1. `CLUSTER MEET B-ip B-port`
2. `CLUSTER MEET C-ip C-port`
3. `CLUSTER MEET D-ip D-port`

由于 `A` 与其他节点互相可见，它会在心跳包中包含gossip部分，让其他节点之间建立连接。即使集群很大，也可以在某一时刻建立起完整的关系网。

而且 `CLUSTER MEET` 不需要双方向交互。如果向A发送命令添加B，不需要再向B发送命令添加A。

## 实现细节：MEET包和PING包

当某节点接收到 `CLUSTER MEET` 消息时，该节点还没有确认命令中所指定的节点。
所以为了让其接受指定节点为受信节点，要使用 `MEET` 包而不是 `PING` 包。
这两个包的格式完全相同，但前者可以强制接收端节点信任指定节点。

@return

@simple-string-reply: 命令成功返回 `OK` ，或指定的地址或端口无效时返回错误。

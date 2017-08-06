返回Redis所有命令的详细信息  @array-reply 。

集群中的客户端必须了解命令中键的位置才能正确执行命令，然而Redis的命令很多，有些是一个键的、有些是多个键甚至有的键会被其他数据分隔开来。

可以使用 `COMMAND` 为每条命令临时保存一个命令与键位置的映射关系，以便准确调度集群实例间的命令。

## 嵌套结果集数组
Each top-level result contains six nested results.  Each nested result is:
顶层的每个结果集包含六个嵌套的结果。每个嵌套的结果是：

  - 命令名称
  - 命令的元数说明
  - 命令标识的嵌套 @array-reply
  - 参数列表中第一个键的位置
  - 参数列表中最后一个键的位置
  - 多个键中每个需要跨越的数量

### 命令名称

命令的名称，以小写字符串表示。

### 命令元数

<table style="width:50%">
<tr><td>
<pre>
<code>1) 1) "get"
   2) (integer) 2
   3) 1) readonly
   4) (integer) 1
   5) (integer) 1
   6) (integer) 1
</code>
</pre>
</td>
<td>
<pre>
<code>1) 1) "mget"
   2) (integer) -2
   3) 1) readonly
   4) (integer) 1
   5) (integer) -1
   6) (integer) 1
</code>
</pre>
</td></tr>
</table>

命令的元数遵循以下简单原则：

  - 如果命令必填参数的数量是固定的，则元数为一个正数。
  - 如果命令必填参数的数量是可变的，则元数为一个负数。

命令的元数 _包含_ 命令名自己。

示例：

  - `GET` 的元数为2，因为命令只接受一个参数且格式固定为 `GET _key_` 。
  - `MGET` 的元数为-2，因为命令接受至少一个参数，多则不限： `MGET _key1_ [key2] [key3] ...` 。

还要注意 `MGET` 中，“最后一个键的位置”的值为-1意思是键的数量可以无限多。

### 标识
命令的标识是一个嵌套 @array-reply 包含一或多个状态值：

  - *write* - 命令可能会引起修改
  - *readonly* - 命令不会改动任何键
  - *denyoom* - 如果发生了OOM则拒绝执行命令
  - *admin* - 服务管理命令
  - *pubsub* - 订阅相关命令
  - *noscript* - 不能在脚本里运行该命令
  - *random* - 返回结果值随机，这样的命令在脚本里使用很危险
  - *sort\_for\_script* - 如果在脚本里执行，则对输出结果排序
  - *loading* - 支持在装载数据库的时候执行
  - *stale* - 支持在复制中有脏数据的时候执行
  - *skip_monitor* - 不在MONITOR中显示
  - *asking* - 集群相关命令 - 即使是导入也会接受
  - *fast* - 命令复杂度介于常数及log(N)之间。用于监控延迟。
  - *movablekeys* - 键的位置不可预知，必须手动检查。


### 可移动的键

```
1) 1) "sort"
   2) (integer) -2
   3) 1) write
      2) denyoom
      3) movablekeys
   4) (integer) 1
   5) (integer) 1
   6) (integer) 1
```

Redis有些命令的键位置不可预知。针对这些命令，将会返回标识 `movablekeys` 。
Redis集群的客户端需要对这些标识为 `movablekeys` 的命令进行手动解析，寻找所有相关键的位置。

目前所有需要手动寻找键的命令有：

  - `SORT` - 可选的 `STORE` 加键, 可选的 `BY` 加权重, 可选的 `GET` 加键
  - `ZUNIONSTORE` - `WEIGHT` 或 `AGGREGATE` 之前的都是键
  - `ZINTERSTORE` - `WEIGHT` 或 `AGGREGATE` 之前的都是键
  - `EVAL` - `numkeys` 数量参数之前的都是键
  - `EVALSHA` - `numkeys` 数量参数之前的都是键

此外 `COMMAND GETKEYS` 是用来获得指定命令语句中键所在位置的命令。

### 参数列表中的键始位置

绝大部分命令第一个键的位置是1。位置0永远是命令名称本身。


### 参数列表中的键终位置

Redis的命令通常接受一个键、两个键，或不限制键的数量上限。

如果命令接受一个键，键始和键终位置是1。

如果命令接受两个键（比如 `BRPOPLPUSH` 、 `SMOVE` 、 `RENAME` 等）键终位置是参数列表中最后一个键的所在位置。

如果命令接受键的数量没有上限，键终位置是-1。


### 跨越数

<table style="width:50%">
<tr><td>
<pre>
<code>1) 1) "mset"
   2) (integer) -3
   3) 1) write
      2) denyoom
   4) (integer) 1
   5) (integer) -1
   6) (integer) 2
</code>
</pre>
</td>
<td>
<pre>
<code>1) 1) "mget"
   2) (integer) -2
   3) 1) readonly
   4) (integer) 1
   5) (integer) -1
   6) (integer) 1
</code>
</pre>
</td></tr>
</table>

通过键的跨越数可以寻找命令如 `MSET` 格式为 `MSET _键1_ _值1_ [键2] [值2] [键3] [值3]...` 的键位置。

在例子 `MSET` 中，每个键的跨越数为2。与其相比 `MGET` 的键跨越数是1。



@return

@array-reply: 命令详细信息的嵌套列表。列表中的命令顺序是随机的。

@examples

```cli
COMMAND
```

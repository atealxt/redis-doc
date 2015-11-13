## EVAL概论

`EVAL` 和 `EVALSHA` 用来使用从2.6.0版本开始编入Redis的Lua解释器来执行脚本。

`EVAL` 的第一个参数是一个Lua 5.1脚本。
在脚本中不需要定义Lua函数（并且也不应该这么做）。
它仅仅是一个在Redis服务环境中运行的Lua程序。

紧随脚本后面 `EVAL` 的第二个参数（总第三个参数）为参数的数量，代表Redis的键的个数。
后续参数可以用一个以一为起始下标的数组由Lua使用全局变量 `!KEYS` 进行访问（如 `KEYS[1]`, `KEYS[2]`, ...）。

所有额外的参数都不应该代表键名，并可以由Lua使用全局变量 `ARGV` 进行访问，方式类似于键（如 `ARGV[1]`, `ARGV[2]`, ...）。

接下来的例子可以诠释上面所讲到的：

```
> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

注意：可以看到Lua数组被返回为Redis multi bulk类型，客户端可以将它转换为自己编程语言的数组类型。

可以在Lua脚本中使用两种Lua函数来调用Redis命令：

* `redis.call()`
* `redis.pcall()`

`redis.call()` 类似于 `redis.pcall()` 。唯一的区别是如果Redis命令执行发生错误， `redis.call()` 会唤起一个Lua错误从而促使 `EVAL` 向命令调用者返回错误，而 `redis.pcall` 会捕获此错误并返回一个包含错误的Lua表结构。

函数 `redis.call()` 和 `redis.pcall()` 的所有参数组成一个完整的Redis命令：

```
> eval "return redis.call('set','foo','bar')" 0
OK
```

上述脚本的作用是设置键 `foo` 的值为字符串 `bar` 。
但它没有遵循 `EVAL` 命令的语义规则，即脚本所使用的键全部都应来自于传递的 `!KEYS` 数组：

```
> eval "return redis.call('set',KEYS[1],'bar')" 1 foo
OK
```

在 `EVAL` 执行前所有的Redis命令都需要进行进行分析以确定命令将对哪些键进行操作。
为了实现 `EVAL` 的这个功能，键也必须明确指定。
这在许多地方都会很有用，特别用于确保Redis集群可以将你的请求转发到相应的集群节点。

注意，为了提供给用户对Redis单实例进行任意配置的机会，此规则并不是强制的，代价则是编写出的脚本无法与Redis集群保持兼容。

Lua脚本可以有一个返回值，通过使用一组转换规则把Lua类型转换成Redis协议中的类型。

## Lua和Redis之间的数据类型转换

当Lua使用call()或pcall()调用Redis命令时Redis的返回值被转换成了Lua的数据类型。
类似当Lua脚本返回值时Lua数据类型被转换成了Redis的数据类型，这样一来脚本就可以将 `EVAL` 的执行结果返回给客户端。

就这种数据类型转换的设计来说，如果Redis类型被转换成Lua类型后再被转换回Redis类型时，结果将与初始值相同。

换句话来说Lua和Redis的类型转换是一对一的关系。
下面的表格展示了所有的转换规则：

**Redis至Lua** 转换表。

* Redis integer reply -> Lua number
* Redis bulk reply -> Lua string
* Redis multi bulk reply -> Lua table (可以嵌套包含其他Redis数据类型)
* Redis status reply -> 包含单个状态信息字段 `ok` 的Lua table
* Redis error reply -> 包含单个错误信息字段 `err` 的Lua table
* Redis Nil bulk reply 与 Nil multi bulk reply -> Lua false boolean 类型

**Lua至Redis** 转换表。

* Lua number -> Redis integer reply （数值被转换成整型值）
* Lua string -> Redis bulk reply
* Lua table (array) -> Redis multi bulk reply （截取至Lua数组中出现的第一个nil值为止）
* 包含单个状态信息字段 `ok` 的Lua table -> Redis status reply
* 包含单个错误信息字段 `err` 的Lua table -> Redis error reply
* Lua boolean false -> Redis Nil bulk reply.

Lua至Redis的转换有一个额外规则，反之没有：

* Lua boolean true -> 值为1的Redis integer reply。

另外有两个重要的规则也需要注意：

* Lua只有一种数字类型，Lua numbers。整型和浮点数之间没有区别。所以我们总是将Lua numbers转换成整型值应答，并移除可能出现的小数部分。**如果想要从Lua中返回浮点数，应返回字符串**，就像Redis自己做的那样（例子见 `ZSCORE` 命令）。
* [通常Lua数组中不能包含nil值](http://www.lua.org/pil/19.1.html)，这是由Lua table的语义决定的，所以当Redis将一个Lua数组转换成Redis协议格式的时候，遇到nil值时转换将停止。

以下是一些转换例子：

```
> eval "return 10" 0
(integer) 10

> eval "return {1,2,{3,'Hello World!'}}" 0
1) (integer) 1
2) (integer) 2
3) 1) (integer) 3
   2) "Hello World!"

> eval "return redis.call('get','foo')" 0
"bar"
```
最后的例子展示了如何在命令执行成功时从Lua的 `redis.call()` 或 `redis.pcall()` 接收确切的返回值。

从下面的例子中可以看出如何处理含有nil值的浮点数或数组：

```
> eval "return {1,2,3.3333,'foo',nil,'bar'}" 0
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) "foo"
```

正如你所见3.333转换成了3，并且由于前面有nil值字符串 *bar* 也是不会返回的。

## 返回Redis类型的辅助函数

这里有两个辅助函数，用于从Lua转换成Redis类型并返回。

* `redis.error_reply(error_string)` 返回一个错误应答。此函数简单的返回具有单字段 `err` 及其描述的表格。
* `redis.status_reply(status_string)` 返回一个状态应答。此函数简单的返回具有单字段 `ok` 及其描述的表格。

使用辅助函数和直接返回指定格式的table没有区别，所以下面两种写法是等价的：

    return {err="My Error"}
    return redis.error_reply("My Error")

## 脚本的原子性

Redis执行所有命令都使用相同的Lua解释器。
Redis保证脚本执行的原子性：当脚本正在执行时不会有其他的脚本或Redis命令能被执行。
这在语义上类似于 `MULTI` / `EXEC` 。
从所有其他客户端的角度来看，脚本的执行结果要么仍然不可见要么已经完成。

然而这也意味着执行缓慢的脚本并不是一个好主意。
创建快速的脚本并不是很难，脚本的开销会非常低。但是如果真的打算使用缓慢脚本的话，应该充分意识到当脚本执行时其它客户端不能执行命令。

## 错误处理

正如前面所讲，调用 `redis.call()` 导致Redis命令错误时将停止执行脚本并返回错误，下面以一种更明显的方式来描述它，让脚本产生一个错误：

```
> del foo
(integer) 1
> lpush foo a
(integer) 1
> eval "return redis.call('get','foo')" 0
(error) ERR Error running script (call to f_6b1bf486c81ceb7edf3c093f4c48582e38c0e791): ERR Operation against a key holding the wrong kind of value
```

使用 `redis.pcall()` 则不会引发错误，而是返回一个在上面提到过的具有特殊格式的错误对象（含有一个字段 `err` 的Lua表结构）。
脚本可以通过 `redis.pcall()` 返回的错误对象，将准确的错误信息返回给用户。

## 带宽与EVALSHA

`EVAL` 命令迫使你一次又一次的重复发送脚本内容。
由于存在内部缓存机制，Redis不需要每次重新编译脚本，所以很多情况下消耗额外带宽并不是一个很好的选择。

另一方面，由于以下原因，使用特殊命令定义命令或配置 `redis.conf` 也将会遇到问题：

*   不同的实例可能有不同的命令实现。

*   当需要确认所有的实例都包含给定的命令时部署将会变得很困难，特别是在分布式环境中。

*   当程序需要调用服务端定义的特殊命令时，命令的语义将很难通过阅读客户端代码而理解。

为了避免这些问题同时避免带宽问题，Redis实现了 `EVALSHA` 命令。

`EVALSHA` 的工作原理与 `EVAL` 基本相同，区别是由脚本的SHA1摘要信息代替脚本自身作为命令的第一个参数。
行为如下：

*   如果服务端能够匹配SHA1摘要信息，则执行相对应的脚本。

*   如果服务端不能匹配SHA1摘要信息，则返回一个特殊错误给客户端并告知使用 `EVAL` 来代替。

例子:

```
> set foo bar
OK
> eval "return redis.call('get','foo')" 0
"bar"
> evalsha 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0
"bar"
> evalsha ffffffffffffffffffffffffffffffffffffffff 0
(error) `NOSCRIPT` No matching script. Please use `EVAL`.
```

客户端可以寄希望于脚本已经保存到了服务器，总是优先使用 `EVALSHA` 来代替 `EVAL`。
如果返回了 `NOSCRIPT` 错误再使用 `EVAL` 。

给 `EVAL` 传递键与其他参数时也同样适用，脚本字符串都会当作常量有效的缓存在Redis中。

## 脚本缓存语义

Redis实例保证对执行过的脚本进行 **永久性** 缓存。
这意味着一旦有Redis实例执行了 `EVAL` 随后所有的 `EVALSHA` 调用都将成功。

脚本可以长时间缓存是因为不像那些复杂的应用程序有很多不同的脚本，不会发生内存不足的问题。
每条脚本就像是一个新命令，即使大型应用程序也就能有几百个而已。
即使多次修改应用程序导致脚本变化，其内存占用也可以忽略不计。

刷新脚本缓存的唯一方法是显式调用 `SCRIPT FLUSH` 命令，它将会对脚本缓存进行 _完整的刷新_ 也就是移除当前为止所有执行过的脚本缓存。

这通常只用在云环境中对其他用户或程序进行实例初始化时。

并且正如之前提到的，脚本缓存没有进行持久化，重启Redis实例会进行清除。
从客户端的角度讲只有两种方法和命令来确认Redis实例是否重启了。

* 与服务器保持的连接没有被关闭。
* 客户端显式使用 `INFO` 命令检查字段 `runid` 的值，确保服务器没有重启，使用的还是同一个进程。

凭经验而论，客户端最好简单地假设脚本已经缓存了，除非管理员手动调用了 `SCRIPT FLUSH` 命令。

在管道环境中，实际上用户可以认为Redis不会删除脚本。

例如当一个程序使用长连接访问Redis时可以确定脚本在执行过一次以后会一直处于内存中，这样在同一任务中继续使用EVALSHA而不是原脚本就不会发生未知脚本错误了（稍候我们将详细阐述这个问题）。

即调用 `SCRIPT LOAD` 加载管道中所有可能出现的脚本，然后使用 `EVALSHA` ，不需要做任何错误处理。

## SCRIPT命令

为了可以控制脚本子系统，Redis提供了SCRIPT命令。
目前SCRIPT接受四种命令：

*   `SCRIPT FLUSH`

    此命令是唯一可以强制Redis刷新脚本缓存的命令。
    这在云环境中非常有用，可以将同一个实例重新指定给一个不同的用户。
    另外也可以用来测试客户端库的脚本功能实现。

*   `SCRIPT EXISTS _sha1_ _sha2_... _shaN_`

    给定一个SHA1摘要列表参数，执行此命令将返回一个1或0的数组，1代表给定的SHA1在缓存中已存在相对应的脚本，而0代表给定的SHA1在缓存中没有相对应的脚本（或至少在执行上一次SCRIPT FLUSH以后没有）。

*   `SCRIPT LOAD _script_`

    此命令用来把给定脚本注册到Redis脚本缓存中。
    适用于任何当我们想要确保 `EVALSHA` 不会执行失败的场合（例如一次管道通信或MULTI/EXEC操作），而不需要真正执行该脚本。

*   `SCRIPT KILL`

    此命令是唯一可以中断处于执行中并且达到配置文件中最长执行时间的脚本的命令。
    SCRIPT KILL命令只应该被用在那些执行过程中不会对数据进行修改的脚本（停止一个只读脚本不会违反脚本引擎所担保的原子性）。
    更多关于长执行时间脚本的信息请见下一节。

## 脚本即函数

编写纯函数脚本是脚本一个非常重要的部分。
在Redis实例中执行的脚本默认会通过发送脚本自身 -- 而不是一个个的命令，复制到从服务器和AOF文件中。

给一个Redis实例发送脚本通常比发送多条命令要快得多，所以如果客户端向主服务器发送很多零散的命令脚本的话，主服务器再把一个个脚本转换成命令，与从服务器交互或存入只可追加文件AOF，会占用过多的带宽流量（同样，在网络传递中进行单个命令调度需要做比通过Lua脚本调度更多的工作，也会引消耗过多的CPU资源）。

通常可以用复制脚本代替脚本执行的语句，但并不总能这样。
所以从Redis 3.2 开始（目前还不是稳定版），脚本引擎可以从执行的脚本中提取写操作序列进行复制，来代替复制脚本自身。
更多信息见下面的章节。
在此章节我们假设脚本以全量脚本进行复制，称之为 **全量脚本复制** 。

这种 *全量脚本复制* 方法主要的缺陷是脚本需要满足如下条件：

* 针对相同的参数、相同的输入数据集，脚本必须总是执行相同的Redis _写_ 命令。
  执行脚本操作不能依赖任何隐藏（非显式）信息，不能依赖在脚本执行过程中或不同的脚本执行之间可能带来的状态改变，也不能依赖任何外部I/O设备的输入。

一些像使用系统时间、调用Redis随机命令如 `RANDOMKEY` 、或使用Lua随机数产生器，都可能导致脚本不是总能执行相同的行为。

为了避免这些问题Redis做了如下处理：

* Lua不执行访问系统时间或其他外部状态的命令。

* 如果脚本在执行了Redis _随机_ 命令如 `RANDOMKEY` 、 `SRANDMEMBER` 、 `TIME` 后，又欲调用可以改变数据集的Redis命令，Redis会终止脚本执行并返回错误。
    这意味着如果脚本只进行只读操作，并不修改数据集的话，这些命令是可以自由使用的。
    注意 _随机命令_ 并不一定是随机数命令：任何具有不确定性的命令都被认为是随机命令（在这点上 `TIME` 命令就是最好的例子）。

* 以随机顺序返回元素的Redis命令如 `SMEMBERS` （因为Redis的集合是 _无序的_ ），在被Lua调用时有着不同于普通调用的行为，会在返回数据给Lua脚本前自动做一次词典排序。
    正由于这样 `redis.call("smembers",KEYS[1])` 总会返回相同顺序的元素，而相同的命令使用普通客户端调用则可能返回不同的结果，即使键包含着完全相同的元素。

* 修改了Lua的伪随机数生成函数 `math.random` 和 `math.randomseed` ，使得每次新脚本的执行都使用相同的种子。
    这意味着每次执行脚本时如果不使用 `math.randomseed` ，调用 `math.random` 总是会生成相同的序数。

而用户仍然可以通过以下简单的方法编写具有随机特性的命令。
假设我要写一个向列表里填充N个随机整数的Redis脚本。

可以从这个简短的Ruby程序开始入手：

```
require 'rubygems'
require 'redis'

r = Redis.new

RandomPushScript = <<EOF
    local i = tonumber(ARGV[1])
    local res
    while (i > 0) do
        res = redis.call('lpush',KEYS[1],math.random())
        i = i-1
    end
    return res
EOF

r.del(:mylist)
puts r.eval(RandomPushScript,[:mylist],[10,rand(2**32)])
```

每次执行脚本返回的结果列表都将是以下元素：

```
> lrange mylist 0 -1
 1) "0.74509509873814"
 2) "0.87390407681181"
 3) "0.36876626981831"
 4) "0.6921941534114"
 5) "0.7857992587545"
 6) "0.57730350670279"
 7) "0.87046522734243"
 8) "0.09637165539729"
 9) "0.74990198051087"
10) "0.17082803611217"
```

为了每次执行脚本可以返回不同的随机元素，同时又保持脚本的纯函数特性，我们可以简单的给脚本添加一个额外参数作为Lua伪随机数生成器的种子。
新的脚本如下：

```
RandomPushScript = <<EOF
    local i = tonumber(ARGV[1])
    local res
    math.randomseed(tonumber(ARGV[2]))
    while (i > 0) do
        res = redis.call('lpush',KEYS[1],math.random())
        i = i-1
    end
    return res
EOF

r.del(:mylist)
puts r.eval(RandomPushScript,1,:mylist,10,rand(2**32))
```

在这里我们把种子作为一个参数发送给伪随机数生成器。
这种情况下给定同样的参数脚本将输出同样的结果，我们增加了一个脚本参数，即在客户端生成的随机种子。
种子参数会在主从复制和只可追加文件中传播，保证相同的变化发生在重新装载AOF或从服务器执行脚本时。

注意：此特性的重要一点是Redis用 `math.random` 和 `math.randomseed` 实现的伪随机数生成器，无论Redis运行在何种架构的系统中都会有着同样的输出。
32位、64位、大端法和小端法的系统都将产生相同的输出。

## 使用命令代替脚本进行复制

从Redis 3.2 开始（目前还不是稳定版）可以选择另一种复制方法。
我们可以只复制脚本生成的单条写命令以替代全量脚本复制，称之为 **脚本效果复制** 。

在此复制模式下，当Lua脚本执行，Redis会收集所有Lua脚本引擎执行的修改数据集的命令。
当脚本执行完成后，脚本生成的命令序列将会以 MULTI / EXEC 事务的形式发送至从服务及AOF中。

这在个别场合中会比较有用：

* 当计算脚本很慢，但执行效果可以转为少量写命令时，不值得再在从服务上或在重新加载AOF时重新计算脚本。
这种情况下复制脚本效果将会好得多。
* 脚本效果复制，不会尝试控制那些无法确认的函数。比如可以自由在脚本的任何地方使用 `TIME` 或 `SRANDMEMBER` 命令。
* 在此模式中，每次调用时Lua PRNG的种子值随机。

为了开启脚本效果复制，需要在执行脚本中的任何写操作前执行以下Lua命令：

    redis.replicate_commands();

如果成功启用了脚本效果复制，方法返回true；否则如果之前脚本已调用了某些写命令，则返回false，继续使用全量脚本复制。

## 选择性复制命令

当开启了脚本效果复制后（见上一章节），可以更进一步地使用命令控制从服务和AOF的复制。
这是一个非常高级的特性，所谓 **过为已甚** ，不当使用可能会打破主从服务以及AOF的强一致逻辑性。

而有时它却很有用，比如只在主服务上执行一些确定的命令创建临时数据。

假设有一个选取两个集合交集的Lua脚本。
从交集中随机选择五个元素建立一个新的集合，最后通过键删除交集集合这个临时数据。
在这里只想复制的是这个有着五个元素的新集合，不需要复制那个临时交集。

因此，Redis 3.2 引入了一个只针对开启脚本效果复制的新命令，可以控制脚本复制引擎。
命令叫做 `redis.set_repl()` ，在关闭了脚本效果复制的情况下调用会发生错误。

命令可以用四种不同的参数调用：

    redis.set_repl(redis.REPL_ALL); -- 复制到AOF和从服务。
    redis.set_repl(redis.REPL_AOF); -- 只复制到AOF。
    redis.set_repl(redis.REPL_SLAVE); -- 只复制从服务。
    redis.set_repl(redis.REPL_NONE); -- 不进行复制。

脚本的默认参数是 `REPL_ALL` 。
通过调用此函数，用户可以开启/关闭AOF和从服务的复制，并随时修改回原来的设置。

一个简单的例子：

    redis.replicate_commands(); -- 启用脚本效果复制。
    redis.call('set','A','1');
    redis.set_repl(redis.REPL_NONE);
    redis.call('set','B','2');
    redis.set_repl(redis.REPL_ALL);
    redis.call('set','C','3');

执行以上脚本，只有键A和C会在从服务和AOF中得到创建。

## 保护全局变量

为了防止数据泄漏至Lua中，Redis脚本不允许创建全局变量。
如果脚本需要在调用时维护状态（非常罕见的需求），应使用Redis键。

如果试图访问全局变量，脚本将停止运行，同时EVAL返回错误：

```
redis 127.0.0.1:6379> eval 'a=10' 0
(error) ERR Error running script (call to f_933044db579a2f8fd45d8065f04a8d0249383e57): user_script:1: Script attempted to create global variable 'a'
```

访问一个 _不存在_ 的全局变量也会产生类似的错误。

使用Lua的调试功能或其他方法比如修改全局保护元数据表来绕过全局保护并不困难。
但实际却很难这么做。
如果用户搅乱了Lua的全局状态，则无法保证AOF和复制的一致性：不要这么干。

给Lua初学者的注意事项：为了防止在脚本中使用全局变量，简单的方法是在声明每个变量时使用 _local_ 关键字。

## 在脚本中使用SELECT

可以像普通客户端那样在Lua脚本中调用 `SELECT` ，Redis 2.8.12和2.8.11在这上面有一点不同。
2.8.12之前Lua脚本中的数据库选择会继续传递并影响脚本调用客户端。
从Redis 2.8.12开始Lua脚本中的数据库选择只会作用于脚本自己，不会影响脚本调用客户端。

之所以在小版本的升级中引入此语义变化，是因为旧的行为会引起Redis复制层的内部数据不一致从而导致缺陷的发生。

## 可用的函数库

Redis的Lua解释器装载了以下Lua函数库：

* `base` lib.
* `table` lib.
* `string` lib.
* `math` lib.
* `struct` lib.
* `cjson` lib.
* `cmsgpack` lib.
* `bitop` lib.
* `redis.sha1hex` function.

每个Redis实例都 _保证_ 包含上面所有的函数库，所以可以确定的是所有的Redis脚本都具有相同的环境。

struct、 CJSON 和 cmsgpack 是外部函数库，所有其他的库都是标准的Lua函数库。

### struct

struct是一个用来打包和拆包数据结构的Lua函数库。

```
合法的格式：
> - 大端法
< - 小端法
![num] - 对齐
x - 边距
b/B - 有符号/无符号的byte
h/H - 有符号/无符号的short
l/L - 有符号/无符号的long
T   - size_t
i/In - 有符号/无符号大小为 `n' 的 integer （默认为int的大小）
cn - `n' 个字符序列（字符串的首尾位置）；在打包时 n==0 的意思是整个字符串；在拆包时 n==0 的意思是使用上一次读到的数作为字符串的长度。
s - 以零结尾的字符串
f - float
d - double
' ' - 忽略
```


例子：

```
127.0.0.1:6379> eval 'return struct.pack("HH", 1, 2)' 0
"\x01\x00\x02\x00"
127.0.0.1:6379> eval 'return {struct.unpack("HH", ARGV[1])}' 0 "\x01\x00\x02\x00"
1) (integer) 1
2) (integer) 2
3) (integer) 5
127.0.0.1:6379> eval 'return struct.size("HH")' 0
(integer) 4
```

### CJSON

CJSON函数库提供非常快速的JSON处理。

例子：

```
redis 127.0.0.1:6379> eval 'return cjson.encode({["foo"]= "bar"})' 0
"{\"foo\":\"bar\"}"
redis 127.0.0.1:6379> eval 'return cjson.decode(ARGV[1])["foo"]' 0 "{\"foo\":\"bar\"}"
"bar"
```

### cmsgpack

cmsgpack函数库提供简单快速的消息包装处理。

例子：

```
127.0.0.1:6379> eval 'return cmsgpack.pack({"foo", "bar", "baz"})' 0
"\x93\xa3foo\xa3bar\xa3baz"
127.0.0.1:6379> eval 'return cmsgpack.unpack(ARGV[1])' 0 "\x93\xa3foo\xa3bar\xa3baz
1) "foo"
2) "bar"
3) "baz"
```

### bitop

Lua的比特操作模块对数字做按位操作。
此功能从Redis 2.8.18开始可用在脚本中。

例子：

```
127.0.0.1:6379> eval 'return bit.tobit(1)' 0
(integer) 1
127.0.0.1:6379> eval 'return bit.bor(1,2,4,8,16,32,64,128)' 0
(integer) 255
127.0.0.1:6379> eval 'return bit.tohex(422342)' 0
"000671c6"
```

除此之外还支持的函数有：
`bit.tobit`, `bit.tohex`, `bit.bnot`, `bit.band`, `bit.bor`, `bit.bxor`,
`bit.lshift`, `bit.rshift`, `bit.arshift`, `bit.rol`, `bit.ror`, `bit.bswap`.
这些函数的详细文档见 [Lua BitOp 文档](http://bitop.luajit.org/api.html)。

### `redis.sha1hex`

对输入字符串执行SHA1。

例子：

```
127.0.0.1:6379> eval 'return redis.sha1hex(ARGV[1])' 0 "foo"
"0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33"
```

## 使用脚本记录Redis日志

可以通过在Lua脚本里使用 `redis.log` 函数向Redis记入日志。

```
redis.log(loglevel,message)
```

`loglevel` 为以下之一：

* `redis.LOG_DEBUG`
* `redis.LOG_VERBOSE`
* `redis.LOG_NOTICE`
* `redis.LOG_WARNING`

这和标准的Redis日志级别是一样的。
只有当脚本的日志级别大于等于当前Redis实例的日志级别配置时，日志才会输出。

参数 `message` 是一个简单的字符串。
例如：

```
redis.log(redis.LOG_WARNING,"Something is wrong with this script.")
```

将会产生如下日志：

```
[32343] 22 Mar 15:21:39 # Something is wrong with this script.
```

## 沙箱及最长执行时间

脚本绝不应该尝试访问外部系统，比如文件系统或其他系统的调用。
脚本只应该被用来操作Redis数据和传递参数。

脚本还有最长执行时间限制（默认为五秒钟）。
通常脚本应该在毫秒单位下执行，默认的超时时间已经是很大的一个数了。
这个限制更多的用来应对那些客户端程序开发中引起的无限循环。

通过配置 `redis.conf` 或使用CONFIG GET / CONFIG SET命令，可以修改脚本的最长执行时间（以毫秒为单位）。
最长执行时间的配置参数为 `lua-time-limit` 。

为了不违反Redis脚本引擎脚本的原子性，当到达超时时间时Redis并不会自动停止执行脚本。
停止脚本意味着可能在数据集中留下写到一半的数据。
因此当脚本超过最长执行时间时将会发生：

* Redis会在日志中对脚本执行时间过长进行记录。
* Redis将开始接收其他客户端的命令，但对绝大多数命令返回BUSY错误。
  只有 `SCRIPT KILL` 和 `SHUTDOWN NOSAVE` 命令可以被正常执行。
* 可以使用 `SCRIPT KILL` 命令来强制停止仅有只读命令的脚本。
  这不会违反脚本约束性，因为没有输据通过脚本写入数据集。
* 如果脚本已经执行了写命令，只能通过 `SHUTDOWN NOSAVE` 命令来停止服务器，放弃保存当前数据集至磁盘（即终止服务器运行）。

## 处于管道中的EVALSHA

在管道的一系列请求上下文中执行 `EVALSHA` 要谨慎，一定要保证好命令按照指定的顺序执行。
如果 `EVALSHA` 返回 `NOSCRIPT` 错误的话，就不能再执行该命令了，否则执行顺序将被打乱。

客户端类库实现应采取以下方法的其中之一：

*   在管道中总是直接使用 `EVAL` 。

*   首先收集所有需要在管道中执行的命令，找到其中所有的 `EVAL` 命令并使用 `SCRIPT EXISTS` 验证其是否已定义。
    如果没有，在管道最初端添加所需的 `SCRIPT LOAD` 命令，再使用 `EVALSHA` 代替所有的 `EVAL` 命令。

对存储在 `键` 中的数值进行自增一操作。
如果键不存在，在执行命令前将其设置为 `0` 。
如果键的值类型错误或类型为字符串但不能转换为整型值时返回错误。
此操作的最大值限制为64位有符号整数。

**注意**：这是一个字符串操作，因为Redis不具有专门的整型类型。
键所存储的字符串被解释为一个10进制的 **64位有符号整数** 并执行操作。

Redis使用自有的整型表示法存储整数，所以对字符串值实际上存储为了一个整型值，不会有额外的将整型表示为字符串的存储开销。

@return

@integer-reply: `键` 自增后的值

@examples

```cli
SET mykey "10"
INCR mykey
GET mykey
```

## 模式：计数器

计数器是使用Redis进行原子自增操作最具代表性的模式。
即每次遇到执行指定操作时简单的向Redis发送一个 `INCR` 命令。

例如一个网络应用程序，我们想知道某个用户在一年中每天进行了多少次的页面访问。

那么网络应用程序可以简单的在每次用户浏览网页时对一个键进行自增，键使用用户ID和代表当前日期的字符串拼接起来。

这个简单的模式可以延伸至多个方向：

* 可以在每次页面浏览时同时使用 `INCR` 和 `EXPIRE` 命令，只对最后N个少于特定秒数间隔的页面浏览进行计数。
* 客户端可以使用GETSET原子性的获得当前计数器的值并重置为零。
* 使用其他原子性的自增/自减命令如 `DECR` 或 `INCRBY` 可以依据用户执行的操作让值变得更大或更小。
  想象一下例如在一个线上游戏中不同用户的分数。

## 模式：频率限制器

频率限制器模式是一种特殊的计数器，通常用来限制某些操作的执行频率。
此模式的经典用例为限制对共有API的请求次数。

我们提供两个使用 `INCR` 实现此模式的例子，我们假设要解决的问题是限制API的调用次数为 _每秒钟每个IP的请求次数最多为十次_ 。

## 模式：频率限制器1

下面是此模式的一个简单直接的例子：

```
FUNCTION LIMIT_API_CALL(ip)
ts = CURRENT_UNIX_TIME()
keyname = ip+":"+ts
current = GET(keyname)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    MULTI
        INCR(keyname,1)
        EXPIRE(keyname,10)
    EXEC
    PERFORM_API_CALL()
END
```

基本来说我们为每一个IP在每秒设置一个计数器。
计数器总是在自增后设置一个10秒钟的超时时间，这样当秒变化时它们会自动被Redis移除。

注意使用 `MULTI` 和 `EXEC` 是为了保证在每次API调用的时候既进行了自增又设置了过期时间。

## 模式：频率限制器2

一种替代的实现是只使用一个计数器，但要做到非竞争会更复杂一些。
我们将对不同的方法进行验证。

```
FUNCTION LIMIT_API_CALL(ip):
current = GET(ip)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    value = INCR(ip)
    IF value == 1 THEN
        EXPIRE(value,1)
    END
    PERFORM_API_CALL()
END
```

这种创建的计数器只提供一秒钟的生存时间，从发起第一个请求的当前秒开始。
如果同一秒钟超过了10次请求，计数器将接收到一个比10大的值，否则计数器将过期并重新从0开始计数。

**上面的代码存在竞争**。
由于一些原因客户端可能会执行 `INCR` 命令但不会执行 `EXPIRE` 命令，键会被遗漏除非再次访问相同的IP。

这个问题可以简单的通过把 `INCR` 和 `EXPIRE` 一起放到Lua脚本中并使用 `EVAL` 执行来解决（只在Redis 2.6版本以后可用）

```
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
    redis.call("expire",KEYS[1],1)
end
```

有另外一种不需要使用脚本的方法能解决此问题，就是使用Redis列表来代替计数器。
这种实现更复杂一些，使用了更多的高级特性，但具有记住执行API调用的客户端IP地址的优点。这是否有用取决于应用程序。

```
FUNCTION LIMIT_API_CALL(ip)
current = LLEN(ip)
IF current > 10 THEN
    ERROR "too many requests per second"
ELSE
    IF EXISTS(ip) == FALSE
        MULTI
            RPUSH(ip,ip)
            EXPIRE(ip,1)
        EXEC
    ELSE
        RPUSHX(ip,ip)
    END
    PERFORM_API_CALL()
END
```

 `RPUSHX` 命令只在键已经存在的情况下压入元素。

注意这里有一个竞争，但这并不是问题： `EXISTS` 可能会返回false，而在 `MULTI` / `EXEC` 块内创建之前，键可能已经由另一个客户端创建了。
但这个竞争只会在极少数情况下丢失一次API调用，所以频率限制仍然会正常的工作。

`MONITOR` 是一个用来传回Redis服务器命令执行情况的调试命令。
它可以帮助了解数据库的运行情况。
此命令既可以通过 `redis-cli` 也可以通过 `telnet` 来执行。

观察服务器的所有处理请求有助于发现应用程序中的错误，无论是把Redis作为数据库还是分布式缓存系统。

```
$ redis-cli monitor
1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
1339518087.877697 [0 127.0.0.1:60866] "dbsize"
1339518090.420270 [0 127.0.0.1:60866] "set" "x" "6"
1339518096.506257 [0 127.0.0.1:60866] "get" "x"
1339518099.363765 [0 127.0.0.1:60866] "del" "x"
1339518100.544926 [0 127.0.0.1:60866] "get" "x"
```

在 `redis-cli` 中使用 `SIGINT` （Ctrl-C） 来停止执行 `MONITOR` 。

```
$ telnet localhost 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
MONITOR
+OK
+1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
+1339518087.877697 [0 127.0.0.1:60866] "dbsize"
+1339518090.420270 [0 127.0.0.1:60866] "set" "x" "6"
+1339518096.506257 [0 127.0.0.1:60866] "get" "x"
+1339518099.363765 [0 127.0.0.1:60866] "del" "x"
+1339518100.544926 [0 127.0.0.1:60866] "get" "x"
QUIT
+OK
Connection closed by foreign host.
```

在 `telnet` 中手动发送 `QUIT` 命令来停止执行 `MONITOR` 。

## 执行 `MONITOR` 的开销

由于 `MONITOR` 监控 **所有** 命令，使用它是有代价的。
以下测试数据（非严谨的）说明了执行 `MONITOR` 可能引起的开销。

在 **没有运行** `MONITOR` 情况下的测试结果：

```
$ src/redis-benchmark -c 10 -n 100000 -q
PING_INLINE: 101936.80 requests per second
PING_BULK: 102880.66 requests per second
SET: 95419.85 requests per second
GET: 104275.29 requests per second
INCR: 93283.58 requests per second
```

在 **运行** `MONITOR` 情况下的测试结果（`redis-cli monitor > /dev/null`）：

```
$ src/redis-benchmark -c 10 -n 100000 -q
PING_INLINE: 58479.53 requests per second
PING_BULK: 59136.61 requests per second
SET: 41823.50 requests per second
GET: 45330.91 requests per second
INCR: 41771.09 requests per second
```

在这个特别的示例中，运行一个 `MONITOR` 客户端可以降低超过50%的吞吐量。
运行更多的 `MONITOR` 客户端将造成更大的影响。

@return

**非标准化返回值**，会一直输出接收到的命令。

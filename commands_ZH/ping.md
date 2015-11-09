如果不提供参数返回 `PONG` ，否则返回参数。
此命令通常用来测试连接是否可用，或测量网络延迟。

如果客户端订阅了某个频道或模式，命令将返回一个数组，第一个位置是 "pong" ，第二个位置是空值。如果提供了参数则直接返回该参数值。

@return

@simple-string-reply

@examples

```cli
PING

PING "hello world"
```

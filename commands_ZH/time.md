 `TIME` 命令返回一个服务器当前时间的列表，包含两个项目：Unix时间戳和当前秒已经过的微秒数。
基本上来说此接口与系统调用 `gettimeofday` 中的一个非常相似。

@return

@array-reply, 具体为：

包含两个元素的multi bulk应答：

* unix时间秒数。
* 微秒数。

@examples

```cli
TIME
TIME
```

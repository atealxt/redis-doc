此命令很像 `DEL` ：删除指定键。
也像 `DEL` 一样，当键不存在时忽略操作。
不同的是此命令会创建一个独立线程对该键的内存进行回收，所以是非阻塞的。 `DEL` 是阻塞的。
这就是此命令名字的来源：只是从键空间中 **删除链接** 。实际的删除操作将会在之后异步进行。 

@return

@integer-reply: 删除链接的键的数量。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
UNLINK key1 key2 key3
```

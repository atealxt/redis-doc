从存储在 `键` 的集合中随机移除并返回一个或多个元素。

 `SRANDMEMBER` 与此操作类似，从集合中随机返回一个或多个元素但并不会进行删除。

参数 `count` 将会在以后的版本中可用，在2.6、2.8和3.0中不可用。

@return

@bulk-string-reply: 返回删除的元素，或当 `键` 不存在时返回 `nil` 。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myset "three"
SPOP myset
SMEMBERS myset
SADD myset "four"
SADD myset "five"
SPOP myset 3
SMEMBERS myset
```

## 指定count的行为说明

如果count大于集合中的元素数量，命令只会返回集合的所有元素，不会额外返回元素。

## 返回元素的分布

注意此命令不适用于需要返回元素均匀分布的场合。更多关于SPOP使用的算法信息，请查阅Knuth和Floyd的抽样算法。

## 关于参数Count

Redis 3.2 将是第一个支持传递可选参数 `count` 至 `SPOP` ，一次性返回多个元素的版本。
它的实现已经放在了分支 `unstable` 中。

命令 `SCAN` 以及其近似命令 `SSCAN` 、 `HSCAN` 和 `ZSCAN` 可以用来增量的迭代元素集合。

* `SCAN` 迭代Redis当前选择的数据库的键集合。
* `SSCAN` 迭代元素集合的值。
* `HSCAN` 迭代哈希表的字段与值。
* `ZSCAN` 迭代有序集合的值与分数。

这些增量迭代的命令每次调用只返回很少数量的元素，可以在生产环境中使用而不会像 `KEYS` 或 `SMEMBERS` 命令那样在调用大量键或元素的集合时长时间（甚至数秒）阻塞服务器。

阻塞命令如 `SMEMBERS` 可以确保在调用时返回的元素一定为集合的一部分，但是SCAN系列命令无法保证这点，在迭代过程中增量迭代集合的元素有可能会改变。

请注意 `SCAN` 、 `SSCAN` 、 `HSCAN` 和 `ZSCAN` 都很像，所以此文档将包括所有这四个命令。
这四个命令的一个明显区别是 `SSCAN`, `HSCAN` 和 `ZSCAN` 的第一个参数为持有该集合、哈希表或有序集合的键。
 `SCAN` 命令并不一定需要传递键参数，在没有传时将迭代当前数据库的所有对象。

## SCAN的基本用法

SCAN是一个基于游标的迭代器。
这意味着每次调用命令时服务器会返回更新后的游标，用户在下次调用时需要将其作为参数传入。

当游标设置为0时代表迭代开始，当服务器返回的游标为0时代表迭代结束。
下面是一个SCAN迭代的例子：

```
redis 127.0.0.1:6379> scan 0
1) "17"
2)  1) "key:12"
    2) "key:8"
    3) "key:4"
    4) "key:14"
    5) "key:16"
    6) "key:17"
    7) "key:15"
    8) "key:10"
    9) "key:3"
   10) "key:7"
   11) "key:1"
redis 127.0.0.1:6379> scan 17
1) "0"
2) 1) "key:5"
   2) "key:18"
   3) "key:0"
   4) "key:2"
   5) "key:19"
   6) "key:13"
   7) "key:6"
   8) "key:9"
   9) "key:11"
```

在上面的例子中，第一次调用使用零作为游标开始迭代。
第二次调用使用的游标是上一次调用的第一个返回元素17。

正如所见 **SCAN的返回值** 是一个包含两个值的数组：第一个值是下次调用时使用的新游标，第二个值是元素数组。

在第二次调用时返回的游标为0，服务器告诉调用者迭代结束，集合已经完成遍历。
以游标0开始迭代，不停的调用 `SCAN` 直到返回的游标为0叫做一次 **完整迭代**。

## Scan 的担保

在迭代过程中 `SCAN` 系列命令可以提供以下几个保证。

* 一次完整迭代从迭代开始到结束会检索集合中的所有元素。这意味着如果在迭代开始时元素在集合中，在迭代结束时它仍然存在的话，会在迭代中的某一次返回给用户。
* 一次完整迭代从迭代开始到结束不会返回在迭代开始前不存在于集合中的元素。在迭代开始前集合中被删除的元素，在整个迭代中是不会被加回来的， `SCAN` 保证不会返回该元素。

然而因为 `SCAN` 只保存着非常少的状态信息（只有游标），它有以下缺点：

* 一个元素可能会被返回多次。重复元素需由应用程序来处理，例如只在重复执行安全的前提下使用返回元素执行后续操作。
* 在一次完整迭代中元素并不一定会出现在集合里，它有可能返回也有可能不会：这是不确定的。

## 每次调用SCAN返回的元素数

`SCAN` 系列命令不保证每次调用时的元素返回数量。
命令也可能会返回零个元素，客户端不应该由此认为迭代结束而应该判断返回的游标是否为零。

这样返回元素是有原因的，实际上SCAN可能在迭代大集合时一次返回数十个元素，或者在迭代小集合时一次性返回所有元素（集合小到可以使用内部的数据结构编码表示如小型集合、哈希和有序集合）。

可以使用选项 **COUNT** 调整返回元素的数量级。

## 选项 COUNT

虽然 `SCAN` 在迭代时不提供返回元素的数量保证，但可以凭借经验使用 **COUNT** 选项调整 `SCAN` 的行为。
基本上COUNT是用户用来指定 *每次调用应该完成从集合中选取元素的工作合计* 。
这 **只是一个暗示** ，通常来说只能寄希望其返回想要的数量。

* COUNT 的默认值为10。
* 当迭代键空间、集合、哈希或有序集合时，数据不会很少以便可以用哈希表表示，假定没有使用 **MATCH** 选项，每次迭代服务通常会返回和 *count* 相等或稍多一点的元素。
* 当迭代的集合编码为了intset（存储整数的小集合），或哈希及有序集合编码为了ziplist（只有少量值的小哈希和集合），通常不管COUNT的值是多少，在第一次调用 `SCAN` 时就会返回所有元素。

重要提示：每次迭代 **不需要使用相同的COUNT值** 。调用者可以根据需要在迭代过程中自由的改变返回结果数，只需在下一次调用命令时传入上一次调用返回的游标。

## 选项 MATCH

通过指定glob风格的模式，可以仅迭代匹配元素，其行为与命令 `KEYS` 唯一的模式参数类似。

只需要在 `SCAN` 命令的末尾添加 `MATCH <模式>` （SCAN系列命令均可）。

下面是一个使用 **MATCH** 迭代的例子：

```
redis 127.0.0.1:6379> sadd myset 1 2 3 foo foobar feelsgood
(integer) 6
redis 127.0.0.1:6379> sscan myset 0 match f*
1) "0"
2) 1) "foo"
   2) "feelsgood"
   3) "foobar"
redis 127.0.0.1:6379>
```

有一点很重要需要注意， **MATCH** 过滤发生在从集合中取得元素之后，即在返回元素至客户端之前。
这意味着如果模式只匹配了集合中很少的元素， `SCAN` 很可能不会返回任何元素。
见下面的例子：

```
redis 127.0.0.1:6379> scan 0 MATCH *11*
1) "288"
2) 1) "key:911"
redis 127.0.0.1:6379> scan 288 MATCH *11*
1) "224"
2) (empty list or set)
redis 127.0.0.1:6379> scan 224 MATCH *11*
1) "80"
2) (empty list or set)
redis 127.0.0.1:6379> scan 80 MATCH *11*
1) "176"
2) (empty list or set)
redis 127.0.0.1:6379> scan 176 MATCH *11* COUNT 1000
1) "0"
2)  1) "key:611"
    2) "key:711"
    3) "key:118"
    4) "key:117"
    5) "key:311"
    6) "key:112"
    7) "key:111"
    8) "key:110"
    9) "key:113"
   10) "key:211"
   11) "key:411"
   12) "key:115"
   13) "key:116"
   14) "key:114"
   15) "key:119"
   16) "key:811"
   17) "key:511"
   18) "key:11"
redis 127.0.0.1:6379>
```

正如所见大部分调用返回的元素数为零，但是最后一次调用使用了 COUNT 1000 ，为的是让命令在迭代中多做一些扫描。

## 多路并行迭代

多个客户端可以同时迭代一个集合。迭代的唯一状态就是游标，会在每一次调用的时候获取并返回客户端。服务端不需要持有任何状态。

## 中途终止迭代

既然服务端不保存状态，唯一的状态就是游标，客户端可以在迭代过程中自由终止而不用向服务器发送任何信号。
启动大量永不结束的迭代不会有任何问题。

## 使用错误的游标调用SCAN

使用错误、负数、越界及其他不正确的游标调用 `SCAN` 将导致不确定的行为但不会引起程序崩溃。 `SCAN` 将不能保证返回元素的正确性。

游标唯一正确的用法是：

* 当开始迭代时游标的值为0。
* 迭代过程中游标的值为上一次SCAN返回的值。

## 结束迭代的担保

 `SCAN` 算法只在迭代集合的边界有最大值的时候保证能够结束，否则迭代会一直进行导致 `SCAN` 无法完成一次完整的迭代。

这显而易见：如果集合不断增长，就会有更多的工作去做以访问所有元素。迭代结束的能力取决于 `SCAN` 的调用次数和选项COUNT的值与集合增长速度的差距。

## 返回值

`SCAN` 、 `SSCAN` 、 `HSCAN` 和 `ZSCAN` 的返回值是一个由两个元素组成的数组，第一个元素是一个字符串表示一个无符号64位整数（游标），第二个元素是一个数组即遍历的元素数组。

* `SCAN` 返回的元素数组是一个键的列表。
* `SSCAN` 返回的元素数组是一个集合的成员列表。
* `HSCAN` 返回的数组的每个元素分别包含两个元素，元素哈希的属性和值。
* `ZSCAN` 返回的数组的每个元素分别包含两个元素，有序集合的成员及关联分数。

## 附加的例子

迭代一个哈希。

```
redis 127.0.0.1:6379> hmset hash name Jack age 33
OK
redis 127.0.0.1:6379> hscan hash 0
1) "0"
2) 1) "name"
   2) "Jack"
   3) "age"
   4) "33"
```

返回或保存存储在 `键` 的 [列表][tdtl] 、 [集合][tdts] 或 [有序集合][tdtss] 包含的元素。
默认情况下以数字方式进行排序，将值转换成双精度浮点数进行比较。
以下是 `SORT` 最简单的描述：

[tdtl]: /topics/data-types#lists
[tdts]: /topics/data-types#set
[tdtss]: /topics/data-types#sorted-sets

```
SORT mylist
```

假设 `mylist` 是一个数字列表，此命令将返回具有元素从小到大排好序的相同列表。
为了从大到小对数字进行排序，请使用 `!DESC` 修饰符：

```
SORT mylist DESC
```

当 `mylist` 包含字符串值，想要以词典顺序进行排序时，使用 `!ALPHA` 修饰符：

```
SORT mylist ALPHA
```

Redis使用UTF-8编码，并假设你正确设置了 `!LC_COLLATE` 环境变量。

可以使用 `!LIMIT` 修饰符来限制返回元素的数量。
修饰符读取 `偏移量` 参数跳过指定的元素数量， `count` 参数为从 `偏移量` 开始返回的元素数量。
下面的例子将返回排序后版本的 `mylist` 的10个元素，从第0个元素开始（ `偏移量` 以零开头）：

```
SORT mylist LIMIT 0 10
```

几乎所有的修饰符可以一起使用。
下面的例子将返回以词典进行倒序排序后的前5个元素：

```
SORT mylist LIMIT 0 5 ALPHA DESC
```

## 以外部键进行排序

有些时候想使用外部键比如权重对元素排序来替代使用列表、集合或有序集合中实际的元素进行排序。
例如列表 `mylist` 包含表示对象唯一ID的元素 `1` 、 `2` 和 `3` ，分别存储于  `对象_1` 、 `对象_2` 和 `对象_3` 。
当这些对象有权重关联在 `weight_1` 、 `weight_2` 和 `weight_3` 时， `SORT` 可以使用它来对 `mylist` 进行排序，语句如下：

```
SORT mylist BY weight_*
```

选项 `BY` 使用模式（例子中为 `weight_*` ）来匹配排序的键。
这些键的名称通过以首次出现的 `*` 替换列表元素实际的值（例子中为 `1` 、 `2` 和 `3` ）来获得。

## 跳过元素排序

选项 `!BY` 也可以用在不存在的键，让 `SORT` 跳过排序操作。
这在想要取得外部键时很有用（见下面的选项 `!GET` ），不需要进行额外的排序。

```
SORT mylist BY nosort
```

## 取得外部键

之前的例子仅仅是返回排好序的ID。
有些时候，取得实际的对象比取得ID更有用（ `object_1` 、 `object_2` 和 `object_3` ）。
使用以下命令可以基于列表、集合或有序集合中的元素取得外部键：

```
SORT mylist BY weight_* GET object_*
```

可以对原始的列表、集合或有序集合中的元素多次使用选项 `!GET` 以取得更多的键。

也可以使用特殊模式 `#` 来 `!GET` 元素本身。

```
SORT mylist BY weight_* GET object_* GET #
```

## 存储SORT操作的结果

默认情况下， `SORT` 返回排好序的元素至客户端。
通过选项 `!STORE` ，可以将结果存储至一个由指定键指向的列表来代替返回至客户端。

```
SORT mylist BY weight_* STORE resultkey
```

一个有趣的模式是，使用 `SORT ... STORE` 并给结果键关联一个 `EXPIRE` 超时设置，这样在应用程序中 `SORT` 的结果就可以缓存一定时间。
其他客户端就可以使用缓存列表来代替每次请求调用 `SORT` 。
当键超时，可以再次调用 `SORT ... STORE` 来创建新版本的缓存。

注意为了正确实现此模式，避免多个客户端在同一时刻重建缓存是很重要的。
这里需要一些锁处理（例如使用 `SETNX` ）。

## 在 `!BY` 和 `!GET` 中使用哈希

可以通过下面的语法使用选项 `!BY` 和 `!GET` 来指向哈希字段：

```
SORT mylist BY weight_*->fieldname GET object_*->fieldname
```

字符串 `->` 用来把键名从哈希字段名中分离出来。
键作为了替代，通过访问结果键取得指定的哈希字段。

@return

@array-reply: 没有 `store` 参数时，命令返回排序后的元素列表。
@integer-reply: 有 `store` 参数时，命令返回目标列表的元素数量。
返回保存地理位置索引的有序集合中的两个成员间的距离。

给定一个存储着地理位置信息的有序集合，其中元素以 `GEOADD` 添加。命令返回以指定单位表示的两个指定成员的距离。

如果未找到其中一个或两个成员，命令返回NULL。

单位必须为以下之一，默认为米：

* **m** 表示米。
* **km** 表示千米。
* **mi** 表示英里。
* **ft** 表示英尺。

计算的距离以地球为完美的球体作为前提，所以可能会有最大0.5%的误差。

@return

@bulk-string-reply, 具体为：

命令返回指定单位的双精度浮点数（以字符串表示）的距离，或当未找到其中一个或两个元素时返回NULL。

@examples

```cli
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEODIST Sicily Palermo Catania
GEODIST Sicily Palermo Catania km
GEODIST Sicily Palermo Catania mi
GEODIST Sicily Foo Bar
```

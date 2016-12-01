返回保存在 *键* 的地理位置索引有序集合中的所有指定成员的位置（经度、纬度）。

给定一个存储着地理位置信息的有序集合，其中元素以 `GEOADD` 添加。通常可以用此命令获得指定成员的坐标。
当通过 `GEOADD` 添加地理位置索引时，坐标会被转为一个52比特的geohash，所以通过此命令获得的坐标可能不完全等于添加时的坐标，有可能会有微小的误差。

命令接收可变数量的参数，所以返回的是个数组，即使只查询一个元素。

@return

@array-reply, 具体为：

命令返回一个数组，其中每个元素都是一个有着两个元素的数组，表示经度与纬度 (x,y) 。 

不存在的元素在数组中以NULL表示。

@examples

```cli
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEOPOS Sicily Palermo Catania NonExisting
```
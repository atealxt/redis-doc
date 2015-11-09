此命令和 `GEORADIUS` 完全一样，除了命令的查询参数不是地区的中心，而是保存地理位置索引的有序集合中的成员名字。

指定成员的地理位置作为查询的中心。

更多关于此命令和参数的信息请见下面的例子，并另请查阅 `GEORADIUS` 文档。

@examples

```cli
GEOADD Sicily 13.583333 37.316667 "Agrigento"
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEORADIUSBYMEMBER Sicily Agrigento 100 km
```

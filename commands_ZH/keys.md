返回所有匹配 `模式` 的键。

此操作的时间复杂度为O(N)，时间消耗为常数并且很短。
例如，运行在一个入门级笔记本电脑中的Redis可以在40毫秒内扫描1百万个键。

**警告**：请考虑把 `KEYS` 当作只运用于生产环境中的一个命令并且小心使用。
在具有大量数据的环境中执行它可能导致严重的性能问题。
此命令被设计用来调试及一些特殊的操作，例如改变键的布局。
不要在正则应用程序代码中使用 `KEYS` 。
如果想在键集合中查找指定键的话，请考虑使用 `SCAN` 或 [sets][tdts]。

[tdts]: /topics/data-types#sets

支持glob风格的模式：

* `h?llo` 匹配 `hello`, `hallo` and `hxllo`
* `h*llo` 匹配 `hllo` and `heeeello`
* `h[ae]llo` 匹配 `hello` 和 `hallo,` 但不匹配 `hillo`

如果想要匹配特殊字符请使用 `\` 来转义。

@return

@array-reply: 匹配 `模式` 的键的列表。

@examples

```cli
MSET one 1 two 2 three 3 four 4
KEYS *o*
KEYS t??
KEYS *
```

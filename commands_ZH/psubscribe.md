让客户端订阅指定模式的频道。

支持glob风格的模式：

* `h?llo` 匹配 `hello`, `hallo` and `hxllo`
* `h*llo` 匹配 `hllo` and `heeeello`
* `h[ae]llo` 匹配 `hello` 和 `hallo,` 但不匹配 `hillo`

如果想要匹配特殊字符请使用 `\` 来转义。

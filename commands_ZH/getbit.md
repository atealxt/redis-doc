返回 _键_ 存储的字符串值所在 _偏移量_ 的比特值。

如果 _偏移量_ 在字符串长度范围之外，则假定字符串是比特值0的连续空间。
如果 _键_ 不存在则假定值为一个空字符串，这样一来 _偏移量_ 则总是会在字符串长度范围之外，值也会被假定为比特值0的连续空间。

@return

@integer-reply: 存储于 _偏移量_ 的比特值。

@examples

```cli
SETBIT mykey 7 1
GETBIT mykey 0
GETBIT mykey 7
GETBIT mykey 100
```

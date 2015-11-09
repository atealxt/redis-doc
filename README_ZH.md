# Redis文档

## 客户端

所有的客户端列表在 `clients.json` 文件里。
JSON对象的一个键对应一个客户端。
例如：

```
"Rediska": {

  # 指定编程语言。
  "language": "PHP",

  # 如果它有自己的网站，放在这里。否则跳过 "url" 键。
  "url": "http://rediska.geometria-lab.net",

  # 代码库的URL。
  "repository": "http://github.com/Shumkov/Rediska",

  # 关于客户端的描述，尽量言简意赅。旨在帮助用户选择他们需要的客户端。
  "description": "A PHP client",

  # 作者与维护人员的Twitter用户名列表。
  "authors": ["shumkov"]

}
```

## 命令

Redis的所有命令都写在文件 `commands.json` 里。

每个命令都对应有一个详细的Markdown描述文件。（**译注：见[命令文件夹](commands_ZH/)**）

为了在Markdown里有更好的可读性，有几点需要注意：

*   文本里出现的命令应使用反撇号括起。例如： `INCR` 。

*   可以用一些特别的关键字来命名Redis中的公共元素。
    例如： `@multi-bulk-reply` 。
    这些关键字会自动展开并链接至相关的文档里。

文档至少需要包含两部分：描述及返回值。
返回值使用关键字 @return 进行标记：

```
返回所有匹配指定模式的键。

@return

@multi-bulk-reply: 所有匹配指定模式的键。
```

**译注：专题文章** <br>
**Redis文档包含许多相关功能的详细介绍，做了一部分中文翻译，见[专题文件夹](topics_ZH/)。**

## 样式指南

请使用以下格式化规则：

* 每行至多80个字符。
* 每个句子另起一行。

幸运的是，本工程使用了一个Markdown自动格式化工具。
为了只格式化修改过的文件，首先使用 `git add` 加入stage状态（这样就不会在出错后丢失修改），然后运行格式化工具：

```
$ rake format:cached
```

格式化工具依赖于：

* Redcarpet
* Nokogiri
* The `par` tool

在Ruby gem中安装：

```
gem install redcarpet nokogiri
```

安装par（OSX）：

```
brew install par
```

安装par（Ubuntu）：

```
sudo apt-get install par
```

## 检查工作

使用Make来检查修改：

```
$ make
```

这可以确保JSON和Markdown通过编译，且所有文本文件中没有拼写错误。

执行这些检查需要安装几个 Ruby gem 和 [Aspell][aspell] 。
这些gem在 `.gems` 文件中，使用下面的命令安装：

```
$ gem install $(sed -e 's/ -v /:/' .gems)
```

拼写检查的例外应加在 `./wordlist` 里面。

[aspell]: http://aspell.net/


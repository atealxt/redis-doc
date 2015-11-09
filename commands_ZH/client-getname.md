 `CLIENT GETNAME` 返回在 `CLIENT SETNAME` 中设置的当前连接名称。由于新的连接没有默认名称，当没有指定的名称时返回null bulk应答。

@return

@bulk-string-reply: 返回连接的名称，或当没有设置名称时返回null bulk应答。

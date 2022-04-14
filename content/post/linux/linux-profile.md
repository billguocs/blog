---
title: "Linux profile优先级"
date: 2022-04-14T23:16:42+08:00
draft: false
tags: [
    "Linux"
]
categories: [
    "Development"
]
---

登入系统获取一个`shell`进程时，读取环境变量有几步：

1. 首先读入全局变量 `/etc/profile`，然后根据其内容读取额外的设定的文档，如 `/etc/profile.d`和`/etc/inputrc`
2. 然后根据不同用户读取`~`路径下`.bash_profile`，如果这读取不了就读取`~/.bash_login`，这个也读取不了才会读取`~/.profile`，这三个文档设定基本上是一样的，读取有优先关系
3. 然后根据用户账号读取`~/.bashrc`

**其他**\
`~/.profile`可以设定本用户专有的路径，环境变量等，它只在登入的时候执行一次\
`~/.bashrc`也是某用户专有设定文档，可以设定路径，设置`alias`，每次`shell script`的执行都会使用它一次

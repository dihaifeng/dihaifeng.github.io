---
title: "nsenter 使用"
date: 2023-07-04T19:41:47+08:00
comment: true
tags: ["Linux"]
categories: ["Liunx"]
---

nsenter是一个命令行工具，可以帮助用户进入指定的Linux命名空间。使用nsenter命令，用户可以进入指定的命名空间，并在其中执行命令。

## 语法

使用nsenter命令的基本语法如下：

```
nsenter [options] [<pid>|<netns>]
```

其中，`<pid>`表示要进入的进程的PID，`<netns>`表示要进入的网络命名空间的路径。常用的选项包括：

- `-t, --target <pid>`：指定要进入的进程PID。
- `-n, --net`：进入网络命名空间。
- `-m, --mount`：进入挂载命名空间。
- `-u, --uts`：进入UTS命名空间。
- `-i, --ipc`：进入IPC命名空间。
- `-p, --preserve-credentials`：保留当前进程的用户和组标识。


## 示例

例如，要进入PID为1234的进程的PID命名空间，可以使用以下命令：

```
nsenter -t 1234 -m -u -i -n /bin/bash
```

该命令将进入PID为1234的进程的命名空间，并在其中启动一个新的bash shell。在该shell中执行的命令将在该命名空间中执行。

进入25722命名空间：
```bash
nsenter -n -t 25722
```


---
title: "hardcode"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: ["文档"]
---


### Shebangs 

```txt
#!/bin/bash -> #!/usr/bin/env bash
#!/usr/bin/python -> #!/usr/bin/env python
```

并非所有系统都有 /bin 目录和 /usr/bin 目录，比较典型的代表加上 NixOS 和 GNU Guix 操作系统。

env 可以根据环境变量 **`$PATH`** 寻找可执行程序（如 bash），比硬编码路径更加灵活，提高可移植性。

不过并非所有脚本都适合使用 env
	- 对安全性要求高的脚本（如以 root 权限运行），硬编码可以避免通过相关 PATH 让 bash 指向恶意程序。
	- 带有参数的 Shebangs 头，如 #!/usr/bin/perl -w 不能改成 #!/usr/bin/env perl -w

[#!/bin/bash 和 #!/usr/bin/env bash 的区别](https://blog.csdn.net/qq_37164975/article/details/106181500)
https://unix.stackexchange.com/questions/29608/why-is-it-better-to-use-usr-bin-env-name-instead-of-path-to-name-as-my


### 可执行程序

#### 判断某个可执行程序是否存在

不规范做法： 

根据硬编码路径判断某个文件是否存在。
判断  `QFile().exists(/usr/bin/Foo)`，

根据 PATH 寻找可执行文件：

```
Exist = !QStandardPaths::findExecutable("Foo").isEmpty();
```

https://github.com/linuxdeepin/dde-network-core/pull/56
https://github.com/linuxdeepin/dde-session-shell/pull/127

#### 执行

Qt 的 QProcess 会自动读取 PATH 环境变量，

QProcess::execute("/usr/bin/touch", QStringList() << sessionCacheFile);
QProcess::execute("touch", QStringList() << sessionCacheFile);

https://github.com/linuxdeepin/deepin-downloader/pull/27

#### 库

/urs/lib 

#### 头文件

绝对不要有 `#include </usr/include/xxx.h>`  这种代码，c/c++ 头文件会自动在 /usr/include 里寻找，直接使用 `#include <xxx.h>` 。

此外，如果库提供了 pkg-config 文件，提供的 -I 参数会指定头文件位置，如果提供 Config.cmake （cmake用）文件，一般会提供  Foo_INCLUDE_DIR 变量，如果提供 .pri （qmake 用）文件，由 QT.foo.includes 提供头文件位置。`xxx.h` 是相对提供的路径寻找。

对于没有提供任何开发文件的库，也可以使用  cmake 提供的 [CheckIncludeFile](https://cmake.org/cmake/help/latest/module/CheckIncludeFile.html) 模块寻找头文件。


###
/usr/share

XDG_DATA_DIRS

相关修改
https://github.com/linuxdeepin/dde-grand-search/pull/30
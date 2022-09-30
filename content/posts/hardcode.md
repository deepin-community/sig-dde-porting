---
title: "尽可能避免使用 hardcode 路径"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "改善可移植性" ]
---

## Shebangs 

```txt
#!/bin/bash -> #!/usr/bin/env bash
#!/usr/bin/python -> #!/usr/bin/env python
```

并非所有系统都有 /bin 目录和 /usr/bin 目录，比如 NixOS 和 GNU Guix 操作系统。
env 可以根据环境变量 **`$PATH`** 寻找可执行程序（如 bash），比硬编码路径更加灵活，可以提高可移植性。

不过并非所有脚本都适合使用 env, 有 2 种情况：
- 对安全性要求高的脚本（如以 root 权限运行），硬编码可以避免通过相关 PATH 让 bash 指向恶意程序。
- 带有参数的 Shebangs 头，如 #!/usr/bin/perl -w 不能改成 #!/usr/bin/env perl -w

对除此之外的大部分的 Shebangs 脚本来说，使用 env 是更好的方案。

参考资料：
- [#!/bin/bash 和 #!/usr/bin/env bash 的区别](https://blog.csdn.net/qq_37164975/article/details/106181500)
- [why-is-it-better-to-use-usr-bin-env-name-instead-of-path-to-name](https://unix.stackexchange.com/questions/29608/why-is-it-better-to-use-usr-bin-env-name-instead-of-path-to-name-as-my)


## 可执行程序

#### 判断某个可执行程序是否存在

不推荐做法：  根据硬编码路径判断某个文件是否存在。比如判断  `QFile().exists(/usr/bin/Foo)`。

推荐做法：根据 PATH 寻找可执行文件, 一般不需要自行读取 PATH，比如 QT 可以使用：

```cpp
Exist = !QStandardPaths::findExecutable("Foo").isEmpty();
```
比如 glib 可以使用 find_program_in_path。

修改示例：
- https://github.com/linuxdeepin/dde-network-core/pull/56
- https://github.com/linuxdeepin/dde-session-shell/pull/127

#### 执行某个可执行程序

同样，推荐用 PATH 寻找，尽量不使用绝对路径。

Qt 的 QProcess 会自动处理 PATH 环境变量，因此 `QProcess::execute("/usr/bin/touch", QStringList() << sessionCacheFile)` 可以直接改成
`QProcess::execute("touch", QStringList() << sessionCacheFile)`。

修改示例：
- https://github.com/linuxdeepin/deepin-downloader/pull/27

#### 动态库路径

尽量不要硬编码 /urs/lib 

qt 应用可以使用 QLibraryInfo::location(QLibraryInfo::LibrariesPath)

### 头文件

绝对不要有 `#include </usr/include/xxx.h>`  这种代码，c/c++ 头文件会自动在 /usr/include 里寻找，直接使用 `#include <xxx.h>` 。

此外，如果库提供了 pkg-config 文件，提供的 -I 参数会指定头文件位置，如果提供 Config.cmake （cmake用）文件，一般会提供  Foo_INCLUDE_DIR 变量，如果提供 .pri （qmake 用）文件，由 QT.foo.includes 提供头文件位置。`xxx.h` 是相对提供的路径寻找。

对于没有提供任何开发文件的库，也可以使用 cmake 提供的 [CheckIncludeFile](https://cmake.org/cmake/help/latest/module/CheckIncludeFile.html) 模块寻找头文件。


### 应用数据（/use/share）

/usr/share，一般保存与架构无关的只读数据。此目录同样不应该硬编码，在 XDG 规范中，这种数据的目录使用环境变量 XDG_DATA_DIRS 设置，如果读不到对应环境变量再去读取 /usr/local/share:/usr/share。

使用 qt 的应用该类型硬编码路径应该使用 QStandardPaths 或者 libqtxdg 代替。

参考：[XDG Base Directory](https://wiki.archlinux.org/title/XDG_Base_Directory)

在此目录寻找某文件，推荐使用 QStandardPaths::standardLocations
参考修改：
- https://github.com/0xd34df00d/leechcraft/commit/32bf44704e156eaf6ada672e18febf64d9fd279f

#### /usr/share/applications

直接使用 `QStandardPaths::standardLocations(QStandardPaths::ApplicationsLocation)`

相关修改：
- https://github.com/linuxdeepin/dde-grand-search/pull/30

#### /usr/share/AppName

读取本应用数据，qt 软件可以使用 `QStandardPaths::standardLocations(QStandardPaths::AppLocalDataLocation)`

相关修改：
- https://github.com/communi/communi-sailfish/commit/16b669653e0e86b24aa4dd20e0434faa07519fb6

#### 导入翻译

部分应用导入翻译使用了 `/usr/share` [示例](https://github.com/linuxdeepin/dde-control-center/blob/ed696dabf41bee19f28758d2589dac20b866c356/src/reset-password-dialog/main.cpp#L63), 应该改为使用 `QStandardPaths::standardLocations(QStandardPaths::GenericDataLocation)`, 此外使用了 dtk 的应用是直接用 [DApplication 的 loadTranslator](https://github.com/linuxdeepin/dtkwidget/blob/f80f48076e1821b06461e1fc330f50ceaff2c812/src/widgets/dapplication.cpp#L776) 导入翻译。

相关 issues：
- https://github.com/linuxdeepin/developer-center/issues/3374
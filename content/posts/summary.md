---
title: "提高可移植性的建议[目录]"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "改善可移植性" ]
weight: 1
---

使用硬编码的路径会因为操作系统的不同而导致可能无法正常使用，比如 Arch Linux 不使用 /usr/libexec （对应文件会放/usr/lib），NixOS 不使用 /lib, /bin, /usr, 等等。  

此外如果使用 linglong 或者 flatpak 等特殊格式打包，同样无法使用传统的硬编码路径。

为了可移植性，应当尽可能避免硬编码。

> 对于安装文件路径，qmake 应尽可能使用 PREFIX 变量而非绝对路径，cmake 除了 CMAKE_INSTALL_PREFIX 外，还提供对于移植非常友好的 GNUInstallDirs 模块，所有的硬编码都应当改用 GNUInstallDirs：

- [安装路径使用 GNUInstallDirs]({{< ref "GNUInstallDirs.md" >}})

> 既然安装路径不同系统有区别，源码中的硬编码自然也需要尽可能避免，遵守 XDG 规范可以避免大量硬编码：
- [源码中的硬编码路径]({{< ref "hardcode.md" >}})



- [规范导出和使用 pkg-config]({{< ref "pkg_config.md" >}})

- [规范导出和使用 Config.cmake]({{< ref "find_package.md" >}})
- [规范版本相关信息]({{< ref "version.md" >}})


> cmake 对一些编译参数有封装，官方的封装有着更好的适用性，比如对不同编译器平台同样功能的参数可能有区别，还可以检查编译器是否支持。

- [规范编译参数]({{< ref "compile_flag.md" >}})

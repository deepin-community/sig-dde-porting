---
title: "规范导出和使用 pkg-config 文件"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "文档" ]
---

## 使用 pkg-config

#### 使用 cmake 生成 .pc 文件

一个比较规范的 foo.pc.in 如下：

```
prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=@CMAKE_INSTALL_FULL_LIBDIR@
includedir=@CMAKE_INSTALL_FULL_INCLUDEDIR@/libimageviewer

Name: foo
Description: foo is a demo lib
Version: @PROJECT_VERSION@
Libs: -L${libdir} -lfoo
Cflags: -I${includedir}
Requires: Qt5Core Qt5Gui Qt5Widgets 
```


这里的 Name 是显示的名称，文件名才是搜索用，显示名一般不要全部大写。

Requires：设置 foo 的依赖， 这里有2个条件：

1. 必须是传播的构建依赖，即某软件构建依赖  foo，构建环境必须有 bar 才可以编译
2. 必须是提供 .pc 文件的依赖

另外不要忘记加 -L 参数（链接库地址），-I 参数（头文件地址）。

相关修改： https://github.com/linuxdeepin/image-editor/pull/27


其中路径问题（libdir/includedir）这里再强调一下， 不规范做法：

```txt
prefix=/usr
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include/foo
```
这种是硬编码路径，可移植性非常差。

```txt
prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include/foo
```
相比上种方法，prefix 使用 cmake 变量，可以应对 prefix 不是 /usr 的情况了。

```txt
prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=${prefix}/@CMAKE_INSTALL_LIBDIR@
includedir=${prefix}/@CMAKE_INSTALL_INCLUDEDIR@/foo
```
相比上种方法，lib/include 目录使用 GNUInstallDirs 变量代替，灵活度更高。

但是，这种并非最好的，因为（1）GNUInstallDirs 有特殊的拼接规则，如 PREFIX=/，LIBDIR=lib，应该拼接成 /usr/include 而非 /include。（2）CMAKE_INSTALL_LIBDIR 虽然默认是相对路径，但允许被直接设置成绝对路径。

写在 .pc.in 只能简单拼接，无法判断特殊情况，最好由 cmake 进行拼接。GNUInstallDirs 提供 FULL 版本变量（本文第一个示例）就是智能拼接出来的，建议使用。当然，基于此自行设置变量也可以：
`set (INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_FULL_INCLUDEDIR}/libdtk-${CMAKE_PROJECT_VERSION}/DCore")`。


#### 使用 qmake 生成 pkgcconfig 文件

```qmake
CONFIG += create_prl create_pc
# 生成 .pc 文件需要 .prl 中间文件，如果不需要可以使用 no_install_prl

QMAKE_PKGCONFIG_NAME = foo
QMAKE_PKGCONFIG_DESCRIPTION = foo is demo
QMAKE_PKGCONFIG_INCDIR = $${HEADERDIR}
QMAKE_PKGCONFIG_LIBDIR = $${LIBDIR}
QMAKE_PKGCONFIG_DESTDIR = pkgconfig
```

其中的 HEADERDIR，LIBDIR 是需要在 .pro 文件中设置的安装路径。

### cmake 使用 pkg-config

不要混淆 find_package 和 pkg_check_modules 

cmake 导入

```
find_package(PkgConfig REQUIRED) # 导入此模块才可以使用 pkg_check_modules
pkg_check_modules(DtkCore REQUIRED dtkcore)
message("${DtkCore_VERSION}")
message("${DtkCore_LIBRARIES}") # 对应 -l 参数
message("${DtkCore_LIBRARY_DIRS}") # 对应 -L 参数
message("${DtkCore_INCLUDE_DIRS}") # 对应 -I 参数

pkg_check_modules(Dtk REQUIRED dtkcore dtkgui dtkwidget)
message("${Dtk_LIBRARIES}")
message("${Dtk_LIBRARY_DIRS}")
message("${Dtk_INCLUDE_DIRS}")
```
参考资料
- https://cmake.org/cmake/help/latest/module/FindPkgConfig.html


### qmake 使用 pkg-config

```
CONFIG += link_pkgconfig
PKGCONFIG += foo
```
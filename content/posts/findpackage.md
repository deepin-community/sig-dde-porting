---
title: "正确导入库文件"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "文档" ]
---

## 正确导入库文件

1. 缺少对依赖的检查，
2. 没有检查依赖的版本
3. 提供


## 使用 pkg-config

```
prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libimageviewer=@CMAKE_INSTALL_FULL_LIBDIR@
includedir=@CMAKE_INSTALL_FULL_INCLUDEDIR@/libimageviewer

Name: foo
Description: deepin image viewer plugins
Version: @PROJECT_VERSION@
Libs: -L${libimageviewer} -limageviewer
Cflags: -I${includedir}
Requires: Qt5Core Qt5Gui Qt5Widgets 
```

这里的 Name 是显示的名称，一般不要全部大写。

Requires：设置 foo 的依赖， 这里有2个条件：

1. 必须是传播的构建依赖，即某软件构建依赖  foo，构建环境必须有 bar 才可以编译
2. 必须是提供 .pc 文件的依赖

相关修改： https://github.com/linuxdeepin/image-editor/pull/27

### 使用 cmake 生成

不规范做法

```
libdir=/usr/lib
includedir=/usr/include/foo

prefix=/usr
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include/foo
```
这两种做法都是硬编码路径，


```
prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include/foo

prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=${prefix}
libdir=${prefix}/@CMAKE_INSTALL_LIBDIR@
includedir=${prefix}/@CMAKE_INSTALL_INCLUDEDIR@/foo
```


```

```



#### 使用 qmake 生成 pkgcconfig 文件

```qmake
CONFIG += create_prl create_pc
# 生成 .pc 文件需要 .prl 中间文件，如果不需要可以使用 no_install_prl

QMAKE_PKGCONFIG_NAME = QMarkdownTextedit
QMAKE_PKGCONFIG_DESCRIPTION = C++ Qt QPlainTextEdit widget with markdown highlighting and some other goodies
QMAKE_PKGCONFIG_INCDIR = $${HEADERDIR}
QMAKE_PKGCONFIG_LIBDIR = $${LIBDIR}
QMAKE_PKGCONFIG_DESTDIR = pkgconfig
```

其中的 HEADERDIR，LIBDIR 是需要在 .pro 文件中设置的安装路径。

### 导入 pkg-config 文件

不要混淆 find_package 和 pkg_check_modules 

cmake 导入

```
find_package(PkgConfig REQUIRED)
find_package(<prefix>
                  <moduleSpec> [<moduleSpec>...])

pkg_check_modules(DtkCore REQUIRED dtkcore)
message("${DtkCore_VERSION}")
message("${DtkCore_LIBRARIES}")
message("${DtkCore_LIBRARY_DIRS}")
message("${DtkCore_INCLUDE_DIRS}")

pkg_check_modules(Dtk REQUIRED dtkcore dtkgui dtkwidget)
message("${Dtk_LIBRARIES}")
message("${Dtk_LIBRARY_DIRS}")
message("${Dtk_INCLUDE_DIRS}")
```

https://cmake.org/cmake/help/latest/module/FindPkgConfig.html




### qmake 导入

```
CONFIG += link_pkgconfig
PKGCONFIG += foo
```



### config.cmake 文件

生成 Config.cmake 文件路径要求与 pkg-cong 一致

应该提供 FooConfig.cmake.in 使用 FULL 版本的 GNUInstallDirs 变量替换路径

相关修改：[use configure_file set path in DdeDockConfig.cmake](https://github.com/linuxdeepin/dde-dock/pull/556/commits/6185843e8ed93c9d22f9921aeefcfa0e73f4f351)



使用 find_dependency 代替 find_package，。

cmake 官方提供了 CMakeFindDependencyMacro 模块，专门用在  Config.cmake 文件中，find_dependency 和 find_package 用法完全相同，因此可以简单的把原来的 find_package 替换成 find_dependency。 

find_dependency 的优点是如果 A  的  


使用 configure_package_config_file 代替 configure_file

提供了 set

 [Installing a Config.cmake file](https://www.f-ax.de/dev/2020/10/07/cmake-config-package.html)

编写自己的 模块

## find_package



相关修改：
https://github.com/linuxdeepin/deepin-calculator/pull/85


使用 dconfig_override_files 请 dtkcommon
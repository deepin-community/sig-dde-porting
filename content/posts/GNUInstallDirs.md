---
title: "在 GNU/Linux 中使用 GNUInstallDirs 优化 cmake 安装路径"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "改善可移植性" ]
---

## 使用 GNUInstallDirs.cmake模块

在指定安装路径时，应当使用变量而非写死安装目录，以便于在不完全符合 FHS 的系统上安装，提高程序的可移植性。对于使用何种变量， GUN 提出了适用于 unix 系统的 [GNU标准安装目录](https://www.gnu.org/prep/standards/html_node/Directory-Variables.html)，GNU/Linux 上使用的就是这套标准的变体。cmake 官方提供了 GNUInstallDirs 模块，定义了一组标准的变量，用于安装不同类型文件到规范指定的目录中。

<!--more-->

要使用这个模块，在 CMakeLists.txt 添加一行 `include(GNUInstallDirs)` 即可导入。如果你发现 *CMAKE_INSTALL_XXXX* 的值为空，大概率是缺少这一行。注意导入模块需要放在使用变量之前。


## 前缀值 CMAKE_INSTALL_PREFIX

CMAKE_INSTALL_PREFIX（**后面简称 PREFIX**） 是一个非常特殊的变量，在 CMakeLists.txt 中所有的相对路径都会默认把 PREFIX 作为前缀进行拼接，组成绝对路径。这一变量是 cmake 基础变量，不导入 GNUInstallDirs 模块也会存在。

举一个例子，假如之前要把文件安装进 /usr/share 目录
```
install(FILES ${QM_FILES} DESTINATION  /usr/share/deepin-calculator/translations)
```
如果设置了 PREFIX=/usr，只需要改成相对路径 share，cmake 会自动拼接成 /usr/share
```
install(FILES ${QM_FILES} DESTINATION  share/deepin-calculator/translations)
```
这样写，如果安装前缀改变，比如改成 `/usr/local`，只需要修改 CMAKE_INSTALL_PREFIX，而不用改 CMakeLists.txt 源码。

目前 CMAKE_INSTALL_PREFIX 按 GNU 的标准默认值是 `/usr/local`。而在 dde 的项目中，一般期望前缀值是 /usr。我们当然可以通过 `cmake -DCMAKE_INSTALL_PREFIX=/usr` 来修改 PREFIX，但最好在项目中添加下面 3 行修改一下 PREFIX 的默认值：

```cmake
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX /usr)
endif ()
```
这里容易犯两种错误：

1. 没有检查 CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT 就直接 set：这会导致用户传入的参数失效，相当于硬编码了路径。
2. 使用 `if (NOT DEFINE CMAKE_INSTALL_PREFIX )` 判断用户是否传参了 PREFIX：实际上 PREFIX 无论什么情况都是有定义的， 只能使用 `CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT` 判断是否是默认值。

对 CMAKE_INSTALL_PREFIX 的配置会对子目录生效，在 cmake 3.14 之前，先应用安装规则再处理 add_subdirectory，而 3.14  及后续版本，会按照声明的顺序运行所有安装规则/add_subdirectory，因此，除非你想给子目录单独配置，上述的设置应写在 add_subdirectory 的前面。 见 [CMP0082](https://cmake.org/cmake/help/latest/policy/CMP0082.html#policy:CMP0082)。

## GNUInstallDirs 变量
在 GNUInstallDirs 中，定义了一些 GNU 标准安装目录的变量，提供给定类型文件的安装路径。这些值可以传递给对应 install() 命令的 DESTINATION 选项。它们通常是相对于安装前缀（PREFIX）的相对路径，以便于以可重定位的方式将其拼接为绝对路径。当然，它们也允许赋值为绝对路径。

### CMAKE_INSTALL_XXXX

- CMAKE_INSTALL_BINDIR
用户可执行程序（ `bin` ）

- CMAKE_INSTALL_SBINDIR
系统管理员可执行程序（ `sbin` ）

- CMAKE_INSTALL_LIBEXECDIR
可执行库文件（ `libexec` ）

- CMAKE_INSTALL_SYSCONFDIR
单机只读数据/read-only single-machine data（ `etc` ）

- CMAKE_INSTALL_SHAREDSTATEDIR
架构无关的可修改数据/modifiable architecture-independent data （ `com` ）

- CMAKE_INSTALL_LOCALSTATEDIR
单机可修改数据/modifiable single-machine data（ `var` ）

- CMAKE_INSTALL_RUNSTATEDIR
3.9版中加入：运行时可修改数据（ `LOCALSTATEDIR/run` ）

- CMAKE_INSTALL_LIBDIR
目标代码库/object code libraries  （ `lib` 或 `lib64` ）
（在 Debian 上当 PREFIX 是 /usr 时，LIBDIR 也可能是 `lib/<multiarch-tuple>` ）

- CMAKE_INSTALL_INCLUDEDIR
C语言头文件（ `include` ）

- CMAKE_INSTALL_OLDINCLUDEDIR
non-gcc 的C语言头文件（ `/usr/include` ）

- CMAKE_INSTALL_DATAROOTDIR
与架构无关的只读数据根目录（ `share` ）

- CMAKE_INSTALL_DATADIR
与架构无关的只读数据（ `DATAROOTDIR` ）

- CMAKE_INSTALL_INFODIR
info 文档（ `DATAROOTDIR/info` ）

- CMAKE_INSTALL_LOCALEDIR
与语言相关的数据/locale-dependent data（ `DATAROOTDIR/locale` ）

- CMAKE_INSTALL_MANDIR
man 文档（ `DATAROOTDIR/man` ）

- CMAKE_INSTALL_DOCDIR
文档根目录（ `DATAROOTDIR/doc/PROJECT_NAME` ）

需要注意的是 DATAROOTDIR 是 DATADIR，LOCALEDIR，MANDIR 和 DOCDIR 共同前缀，不应该在 install 中直接使用 `DATAROOTDIR` 作为参数，而是应该使用 `DATADIR` 代替 。类似 /usr/share/man 的路径应该用 `MANDIR` 代替，而不是 `DATADIR/man`。

### CMAKE_INSTALL_FULL_XXXX
CMAKE_INSTALL_BINDIR 推荐用相对路径，但也可以使用绝对路径，有些时候我们需要绝对路径（比如生成 pkgconfig 文件时），直接加 PREFIX 拼接并不好（因为 BINDIR 可能已经是绝对路径了），这时，我们可以直接使用 `CMAKE_INSTALL_FULL_BINDIR`，这是由 GNUInstallDirs 模块提供，自动从 BINDIR 计算出的绝对路径。如果 BINDIR 是绝对路径，直接相等。如果 BINDIR 是相对路径，则等于 BINDIR 按照一定规则与 PREFIX 拼接而成的值。

用例： https://github.com/linuxdeepin/dde-dock/pull/556

当然，install 是没有必要用 CMAKE_INSTALL_FULL_XXXX 的（当然用也可以），因为它会自动判断，并拼接相对路径。

## 前缀拼接的特殊情况

GNUInstallDirs 前缀拼接存在一些特殊情况需要注意：

#### 1. CMAKE_INSTALL_PREFIX=/

除了 `SYSCONFDIR`, `LOCALSTATEDIR` 和 `RUNSTATEDIR` 正常拼接外, 其他值都会增加一个 usr/  前缀 。比如 `INCLUDEDIR` 默认值 include ，变成 usr/include, 最终拼接成 /usr/include。

```
这里自动增加 usr 是符合 GNU 目录标准的，因为这些路径是符号链接。
~ ❯❯❯ readlink /lib
usr/lib
~ ❯❯❯ readlink /bin
usr/bin
```

#### 2. CMAKE_INSTALL_PREFIX=/usr

`SYSCONFDIR`, `LOCALSTATEDIR` 和 `RUNSTATEDIR` 拼接时只拼接 “/”。 比如,  `SYSCONFDIR` 默认值 etc 会拼接成 /etc。

#### 3. CMAKE_INSTALL_PREFIX=/opt/...

`SYSCONFDIR`, `LOCALSTATEDIR` 和 `RUNSTATEDIR` 会向后拼接。 比如， `SYSCONFDIR` 会变成 /etc/opt/.... 


## 参考文档
- [GNUInstallDirs 文档](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html)
- [cmake install 文档](https://cmake.org/cmake/help/latest/command/install.html)
- [GNUInstallDirs 源码](https://github.com/Kitware/CMake/blob/master/Modules/GNUInstallDirs.cmake)



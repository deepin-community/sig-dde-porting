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

## 使用 Config.cmake 文件
### 生成 Config.cmake 文件

生成 Config.cmake 文件路径要求与 pkg-cong 一致

#### 应该提供 FooConfig.cmake.in 使用 FULL 版本的 GNUInstallDirs 变量替换路径

相关修改：[use configure_file set path in DdeDockConfig.cmake](https://github.com/linuxdeepin/dde-dock/pull/556/commits/6185843e8ed93c9d22f9921aeefcfa0e73f4f351)


#### 使用 find_dependency 代替 find_package

cmake 官方提供了 CMakeFindDependencyMacro 模块，专门用在  Config.cmake 文件中，find_dependency 和 find_package 用法完全相同，因此可以简单的把原来的 find_package 替换成 find_dependency。 

find_dependency 的优点是如果 A 的 Config.cmake 寻找 B 的，如果失败 find_package 只会提示 B 的错误，而 find_dependency 还在报错中输出调用链，清晰显示是谁在找 B。

find_dependency 应该在 set 路径之后。

与 pkg-config 的 Requires 类似，这里只 find 传播的构建依赖。


#### 使用 configure_package_config_file 代替 configure_file

使用 FULL 版本的 GNUInstallDirs 变量虽然正确，但还有一个缺陷，生成的是绝对路径，库必须安装到对应目录才可以。而 configure_package_config_file 可以自动计算相对的路径，适用性更广泛。

```cmake
configure_package_config_file(misc/PkgNameConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/PkgNameConfig.cmake
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/PkgName"
    PATH_VARS INCLUDE_INSTALL_DIR TOOL_INSTALL_DIR
)
```

INSTALL_DESTINATION 是 Config.cmake 的安装路径，但这里并不会安装，只是用来计算相对路径的，所以 install 还是需要写的。

PATH_VARS 传入 PkgNameConfig.cmake.in 里会用到的路径变量。

为了使用 configure_package_config_file，Config.cmake.in 也需要做出以下变化：

- 在开头写一行 @PACKAGE_INIT@，这会展开生成 CMakePackageConfigHelpers 提供的宏，包括set_and_check，check_required_components。

- 使用 set_and_check 替代 set 设置路径，可以在引入时检查对应的路径文件是否存在。（注意如果拆包了其他包的文件就不要 check 了）。

- 路径变量前面加 PACKAGE，如 @INCLUDE_INSTALL_DIR@ 改成 @PACKAGE_INCLUDE_INSTALL_DIR@，需要上面 configure_package_config_file 的 PATH_VARS 参数传进来。

- 文件后面使用 check_required_components(PkgName) 检查组件，即使没有拆分组件也应该检查，明确告诉 cmake 该包没有组件。

#### 使用 Targets

对于新增的库，建议使用新式 cmake 写法，使用 Targets，而非手动设置路径：

建议参考： [Installing a Config.cmake file](https://www.f-ax.de/dev/2020/10/07/cmake-config-package.html)


## 使用 find_package 引入库

find_package 有两个模式，Modules Mode 和 Config Mode，在 Modules Mode下，find_package(PkgName) 会寻找命名为 FindPkgName.cmake 的模块，由此模块负责寻找，这种模块一般是由 cmake 官方提供或者调用者自己编写。Config Mode 就是寻找 PkgNameConfig.cmake 模块了，由库自己提供。

```cmake
find_package(DtkCore)
message("${DtkCore_VERSION}") # 6.2 以前没有提供
message("${DtkCore_LIBRARIES}")
message("${DtkCore_LIBRARY_DIRS}") # ！没有提供
message("${DtkCore_INCLUDE_DIRS}") # ！没有提供

find_package(Dtk REQUIRED Core Gui Widget)
# 与 pkg_check_modules 不同，后面是同一（组）软件提供的不同 components，等价于：
find_package(Dtk)
find_package(DtkCore)
find_package(DtkGui)
find_package(DtkWidget)
```

find_package 提供的变量请检查 Config.cmake 提供了什么，不像 pkg_check_modules 那样命名统一，在 6.2 以前，提供 DTKCORE_INCLUDE_DIR， 6.2 之后使用 DtkCore_INCLUDE_DIR（旧版变量仍保留）。

使用 find_package 引入 dtk 的软件，如果用了 DtkCore_INCLUDE_DIRS 请修改，这是 pkg_check_modules 引入才有的变量，另外 DTKCORE_INCLUDE_DIR 升级后 建议换成 DtkCore_INCLUDE_DIR。Gui，Widget 同理。

参考资料：
- https://cmake.org/cmake/help/latest/command/find_package.html
相关修改：
- https://github.com/linuxdeepin/deepin-calculator/pull/85

### 编写 FindXXX.cmake 模块

对于没有提供 pkg-config 也没有提供 Config.cmake 并且 cmake 官方没有提供对应模块的，建议自行写一个 FindXXX.cmake 模块

1. 通过 find_path，find_library 函数计算出 foo_INCLUDE_PATH, foo_LIBRARY。

2. 通过 FindPackageHandleStandardArgs 模块提供的 find_package_handle_standard_args 根据 foo_INCLUDE_PATH, foo_LIBRARY 是否有值设置 foo_FOUND，这样是让 find_package 识别此模块所需要的。

3. 一般会把 foo_INCLUDE_PATH, foo_LIBRARY 标记为高级（mark_as_advanced），不再 cmake gui 显示。

编写模块自由度很高，还可以根据需求检查 check_function_exists，check_symbol_exists 影响 foo_FOUND 而且只需要编写一次，其他项目可以直接复用。

修改示例：
 - https://github.com/linuxdeepin/image-editor/pull/28

##### dtkcommon 相关

使用 dconfig_override_files 请不要忘记通过 `find_package(Dtk)
` 引入 dtkcommon 提供的模块，而不是靠 DtkWidget 的传播依赖（实际上不应该传播 dtkcommon 的）。
---
title: "正确设置版本信息"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: ["文档"]
---

### VERSION

第一位不兼容（可能包含接口删除），第二位（新增接口），第三位（无接口变化，问题修复）


对于供他人调用的库（尤其是活跃开发中，接口经常变化的）来说，版本信息非常重要



版本号可以确定一个软件的功能，

比如，某个开发库 A 新增了一个头文件，在升级了这个库依赖后引入了这个头文件。外部开发者想要编译这个软件, 最好提只能得到 “xxx.h not found” 的编译错误。 如果头文件命名和库名有关还好点，很多时候根本看不出来到达是那里出错了，是自己的环境被破坏了，是自己安装的某个依赖版本太高了还是太低了，如果是，是具体哪个依赖，正确的版本又是什么。这需要花费很多时间才能解决。如果加上版本检查，在一开始就会提示 Could not find a configuration file for package "FOO" that is compatible with requested version "1.2"


### 设置正确版本

```
set (VERSION "1.0.20" CACHE STRING "define project version")
```

版本默认值最好


设置 project 

```
project (libimageviewer
  VERSION ${VERSION}
  DESCRIPTION "libimageviewer is a public library for deepin-image-viewer and deepin-album"
  HOMEPAGE_URL "https://github.com/linuxdeepin/image-editor"
  LANGUAGES CXX C
)
```

不要有类似 `project(deepin-camera VERSION 1.0.0)` 的写法了

提供 project 配置好版本后 cmake 会自动提供  PROJECT_VERSION，PROJECT_VERSION_MAJOR，PROJECT_VERSION_MINOR，PROJECT_VERSION_PATCH 和
PROJECT_VERSION_TWEAK

```
set_target_properties(${TARGET_NAME} PROPERTIES
                    VERSION "${PROJECT_VERSION}"
                    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
```

#### 确保 .pc 文件中的版本

确保 pkgconfig 文件提供正确的版本号：

```
Version: @PROJECT_VERSION@
```

提过 configure_file 

##### 引入时检查

在 cmake 中使用 pkg_check_modules 可以非常方便的判断引入包的版本，直接使用   =, <, >, <= 或者 >= 就可以。

```
pkg_check_modules (GLIB2 glib-2.0) # 默认允许任何版本
pkg_check_modules (GLIB2 glib-2.0>=2.10) # 需要 2.10 及以上版本
pkg_check_modules (FOO glib-2.0>=2.10 gtk+-2.0) # 需要 2.10 及以上版本的 glib-2.0 和任意版本的 gtk+-2.0
```

另外，当 moduleSpec 只有一个时，FindPkgConfig 模块还会提供 <XXX>_VERSION 变量。

```
pkg_check_modules (GLIB2 glib-2.0)
message("${GLIB2_VERSION}")  # 这里会输出找到的 glib-2.0 版本值，如 2.58.3
```

参考： https://cmake.org/cmake/help/latest/module/FindPkgConfig.html

#### 提供 cmake 版本文件 

如果项目提供了 PkgNameConfig.cmake 用于 cmake find_package导入，应该同时提供版本信息（文件命名为 PkgNameConfigVersion.cmake）供调用者检查。

cmake 有一个专门用来生成 Config.cmake 的模块，可以通过  `include(CMakePackageConfigHelpers)` 引入。该模块提供了 write_basic_package_version_file 函数，可以直接生成 ConfigVersion.cmake 文件。使用示例如下：

```
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/PkgNameConfigVersion.cmake"
    VERSION ${DVERSION}
    COMPATIBILITY SameMajorVersion
)
```

- 文件名，必须以 PkgNameConfigVersion.cmake 格式命名，find_package 可以自动失败。
- VERSION 传入当前项目的版本号，缺省值是 PROJECT_VERSION。
- COMPATIBILITY 版本检查策略，有  AnyNewerVersion，SameMajorVersion，SameMinorVersion，ExactVersion 四种。AnyNewerVersion 版本检查最为宽松，只需要提供的版本比需求的版本要相同或者更新，如需要 1.2 版本，找到了 1.2，1.5，1.99 或者 2.0 都是合法的，而找到 1.1 版本将视为没有找到这个包。而 SameMajorVersion 还要求主版本号也必须相同，前面的例子中，要求 1.2 找到 2.0 版本会被视为寻找失败 。类似的 SameMinorVersion 要求前 2 位版本号必须一致。ExactVersion 要求版本号完全一致，不考虑向后兼容的软件需要用这个。一般来说，DDE 大部分项目应该使用 SameMinorVersion。


生成的  PkgNameConfigVersion.cmake 文件应该安装到和 PkgNameConfig.cmake 同一个目录中，一般这个目录是 `${CMAKE_INSTALL_LIBDIR}/cmake/PkgName`


##### 引入包时检查版本

find_package 支持 version 参数，

```
find_package(DtkGui 5.6.0 REQUIRED)
```

```
find_package(Qt5Gui 5.10 REQUIRED)
message("${Qt5Gui_VERSION}")   # 输出找到的版本，如 5.15.3
```

对于可以多版本共存的包，默认会导入第一个找到的，如果需要找最高版本的，可以设置 `set(CMAKE_FIND_PACKAGE_SORT_ORDER NATURAL)`。

参考资料： https://cmake.org/cmake/help/latest/command/find_package.html


修改参考：
https://github.com/linuxdeepin/dtkgui/pull/61
https://github.com/linuxdeepin/image-editor/pull/28


参考资料：

https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION.html
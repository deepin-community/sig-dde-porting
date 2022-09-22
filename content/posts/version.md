---
title: "合理利用开发文件的版本信息"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: ["文档"]
---

对于供他人调用的库（尤其是活跃开发中，接口经常变化的）来说，版本信息非常重要。版本变化意味这接口变化。应用引入库时也应该检查版本号，方便其他人编译。比如，某个开发库 A 新增了一个头文件，在升级了这个库依赖后引入了这个头文件。外部开发者想要编译这个软件, 最后只能得到 “xxx.h not found” 的编译错误。 如果头文件命名和库名有关还好点，很多时候根本看不出来到达是那里出错了，是自己的环境被破坏了，还是自己安装的某个依赖版本太高了还是太低了，如果是，是具体哪个依赖，正确的版本又是什么。这需要花费很多时间才能解决。如果加上版本检查，在编译开始之前就可以发现问题。


<!--more-->

## 设置正确版本

```cmake
set (VERSION "5.5" CACHE STRING "define project version")
```

cmake 版本号格式是 `<major>[.<minor>[.<patch>[.<tweak>]]]`，最高 4 位的版本号，参见 [CMP0048](https://cmake.org/cmake/help/latest/policy/CMP0048.html#policy:CMP0048) 。一般来说，major version 改变意味着不再保证兼容（可能包含接口删除），minor version 改变可能有新增接口，patch version 改变应该无接口变化，只是 bug 修复等。

设置版本号时应该使用 CACHE，这允许编译时使用 -DVERSION 赋值, 默认值也不要写死 1.0.0, 建议默认值至少让 minor version 应该保持正确（对库的要求应该比对应用的要求更严格）。

此外，编译时 -DVERSION 不能直接使用 DEB_VERSION_UPSTREAM，因为 deb 的版本号是可以类似 `1.0.2~2` 这样带有 '~' 或者 '-' 符号的，这不符合 cmake 标准，应该处理一下：
`PACK_VER = $(shell echo $(DEB_VERSION_UPSTREAM) | awk -F'[+_~-]' '{print $$1}')`

参考修改：[fix: deb version incompatible cmake standard](https://github.com/linuxdeepin/image-editor/pull/28/commits/7e7ad599f5d1fd8410ae1f032f1881234b1b6eae)


### 设置 project 

project 可以传入 VERSION 参数，不要使用 `project(deepin-camera VERSION 1.0.0)` 这种写死的错误版本号。

```cmake
project (foo
  VERSION ${VERSION}
  DESCRIPTION "foo is a demo library"
  HOMEPAGE_URL "https://github.com/ownername/foo"
  LANGUAGES CXX C
)
```

project 配置正确后 cmake 会自动获得 `PROJECT_VERSION`，`PROJECT_VERSION_MAJOR`，`PROJECT_VERSION_MINOR`，`PROJECT_VERSION_PATCH` 和
`PROJECT_VERSION_TWEAK` 变量，可以自由使用，比如：

```cmake
set_target_properties(${TARGET_NAME} PROPERTIES
                    VERSION "${PROJECT_VERSION}"
                    SOVERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
```
## pkg-config 相关设置
### [库] 确保 .pc 文件中的版本

确保 pkgconfig 文件提供正确的版本号，在 foo.pc.in 文件中，版本号用：

```txt
Version: @PROJECT_VERSION@
```

之后使用 `configure_file` 就可以替换出正确的版本号了。

### [应用] 引入包时检查版本

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

## Config.cmake 相关设置
### [库] 提供版本文件 

如果项目提供了 PkgNameConfig.cmake 用于 cmake find_package 导入，应该同时提供版本信息（文件命名为 PkgNameConfigVersion.cmake）供调用者检查。

cmake 有一个专门用来生成 Config.cmake 的模块，可以通过  `include(CMakePackageConfigHelpers)` 引入。该模块提供了 write_basic_package_version_file 函数，可以直接生成 ConfigVersion.cmake 文件。使用示例如下：

```cmake
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/PkgNameConfigVersion.cmake"
    VERSION ${DVERSION}
    COMPATIBILITY SameMajorVersion
)
```

- 文件名，必须以 PkgNameConfigVersion.cmake 格式命名，find_package 可以自动识别。
- VERSION 传入当前项目的版本号，缺省值是 PROJECT_VERSION。
- COMPATIBILITY 版本检查策略，有  AnyNewerVersion，SameMajorVersion，SameMinorVersion，ExactVersion 四种。AnyNewerVersion 版本检查最为宽松，只需要提供的版本比需求的版本要相同或者更新，如需要 1.2 版本，找到了 1.2，1.5，1.99 或者 2.0 都是合法的，而找到 1.1 版本将视为没有找到这个包。而 SameMajorVersion 还要求主版本号也必须相同，前面的例子中，要求 1.2 找到 2.0 版本会被视为寻找失败 。类似的 SameMinorVersion 要求前 2 位版本号必须一致。ExactVersion 要求版本号完全一致，不考虑向后兼容的软件需要用这个。一般来说，DDE 大部分项目应该使用 SameMinorVersion。


生成的  PkgNameConfigVersion.cmake 文件应该安装到和 PkgNameConfig.cmake 同一个目录中，一般这个目录是 `${CMAKE_INSTALL_LIBDIR}/cmake/PkgName`

### [应用] 引入库时检查版本

find_package 支持 version 参数：

```cmake
find_package(DtkGui 5.6.0 REQUIRED)
```

```cmake
find_package(Qt5Gui 5.10 REQUIRED)
message("${Qt5Gui_VERSION}")   # 输出找到的版本，如 5.15.3
```

对于可以多版本共存的包，默认会导入第一个找到的，如果需要找最高版本的，可以设置 `set(CMAKE_FIND_PACKAGE_SORT_ORDER NATURAL)`。

参考资料：
- https://cmake.org/cmake/help/latest/command/find_package.html
- https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION.html

参考修改：
- https://github.com/linuxdeepin/dtkgui/pull/61
- https://github.com/linuxdeepin/image-editor/pull/28

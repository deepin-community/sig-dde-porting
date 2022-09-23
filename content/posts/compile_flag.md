---
title: "规范设置编译参数"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: [ "文档" ]
---

CMake 中有许多关于编译器和链接器的设置。当你需要添加一些特殊的需求，你应该首先检查 CMake 是否支持这个需求，如果支持的话，你就可以不用关心编译器的版本，一切交给 CMake 来做即可。 更棒的是，你可以在 `CMakeLists.txt` 表明你的意图，而不是通过开启一系列标志 (flag) 。

<!--more-->

## 优化参数(-O0/-O1/-O2/-O3/-Ofast/-Os) 和 debug 参数 （-g）

CMAKE_BUILD_TYPE 包括 `Debug`, `Release`, `RelWithDebInfo` 和 `MinSizeRe` 4 个选项

它们对应参数是（不同编译平台可能有少许区别）：

1. Release: `-O3 -DNDEBUG`
2. Debug: `-O0 -g`
3. RelWithDebInfo: `-O2 -g -DNDEBUG`
4. MinSizeRel: `-Os -DNDEBUG`

这些参数可以通过 CMAKE_CXX_FLAGS_DEBUG，CMAKE_CXX_FLAGS_RELEASE... 查看，并不直接体现在 CMAKE_CXX_FLAGS 中。

CMAKE_BUILD_TYPE 的默认值不是Release，也不属于上面4种，而是空值，即不附加任何参数。

#### 全局（或非debug模式）设置 -O3：
建议改为将 CMAKE_BUILD_TYPE 设置默认 Release
- https://github.com/ros-infrastructure/bloom/issues/327

#### Release 模式加 O3, Debug 模式加 -g 等等
不用加，这些都是无用功。 

#### Release 模式加 -g
如果软件发行版本需要 debug 符号，请用 RelWithDebInfo 模式

#### Debug 加 -O3
高级别的优化会严重影响 debug 调试， 可以改用 -Og，它允许不影响调试的优化

#### 同时有 -O1 -O3
-O0/-O1/-O2/-O3/-Ofast 开启的优化参数前依次递增，是完全的子集关系，不是多多益善的。

-Os 是在 -O2 的基础上，尽可能优化程序大小。

参考资料： 
- https://stackoverflow.com/questions/48754619/what-are-cmake-build-type-debug-release-relwithdebinfo-and-minsizerel


## c++标准版本 (比如 -std=c++11)

直接使用 set(CMAKE_CXX_STANDARD 11)

部分项目设置了 CMAKE_CXX_STANDARD 为 14/17, 又加了参数 -std=c++11，这是冲突的，请检查。
比如： https://github.com/linuxdeepin/deepin-movie-reborn/blob/0a08e29e78c4f35f787b999d857f696f804f8641/CMakeLists.txt#L18

## as-need 参数

已知 as-need 参数 break 了 mold 连接器，可用 as-needed 代替。
相关：https://github.com/linuxdeepin/developer-center/issues/3345

##  dl 库

如果你需要链接到 `dl` 库，在 Linux 上可以使用 `-ldl` 标志，不过在 CMake 中只需要在 `target_link_libraries` 命令中使用内置的 CMake 变量 [`${CMAKE_DL_LIBS}` ](https://cmake.org/cmake/help/latest/variable/CMAKE_DL_LIBS.html)。这里不需要模组或者使用 `find_package` 来寻找它。（这个命令包含了调用 `dlopen` 与 `dlclose` 的一切依赖）

## 程序间优化(Interprocedural optimization)

[`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html)，最有名的是 *链接时间优化* 以及 `-flto` 标志，这在最新的 CMake 版本中可用。你可以通过变量 [`CMAKE_INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest//CMAKE_INTERPROCEDURAL_OPTIMIZATION.html)（ CMake 3.9+ 可用）或对目标指定 [`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html) 属性来打开它。在 CMake 3.8 中添加了对 GCC 及 Clang 的支持。如果你设置了 `cmake_minimum_required(VERSION 3.9)` 或者更高的版本（参考 [CMP0069](https://cmake.org/cmake/help/latest/policy/CMP0069.html)），当在编译器不支持 [`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html) 时，通过变量或属性启用该优化会产生报错。你可以使用内置模块 [CheckIPOSupported](https://cmake.org/cmake/help/latest/module/CheckIPOSupported.html) 中的 `check_ipo_supported()` 来检查编译器是否支持 IPO 。下面是基于 CMake 3.9 的一个例子：

```cmake
include(CheckIPOSupported)
check_ipo_supported(RESULT result)
if(result)
  set_target_properties(foo PROPERTIES INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()
```

## -lpthread

POSIX thread  是基于 C/C++ 的标准线程库。在 cmake 3.1 以上的版本提供了 FindThreads 模块， 如果设置 THREADS_PREFER_PTHREAD_FLAG 变量，会优先使用 pthread。

```cmake
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(my_app PRIVATE Threads::Threads)
```

- https://cmake.org/cmake/help/latest/module/FindThreads.html
- https://stackoverflow.com/questions/1620918/cmake-and-libpthread/29871891

参考文档：

https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/chapters/features/small.html


## 设置时不要覆盖之前的选项

错误示例：set(CMAKE_CXX_FLAGS "-Wall")
正确示例：set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")

如果确实有覆盖的要求，请在源码注释说明，因为一般出现覆盖 flag 很可能是失误。

## 判断架构

遗留代码有使用 dpkg-architecture 判断的，改用 CMAKE_SYSTEM_PROCESSOR。

dpkg-architecture 是 debian 系特有的，并不通用。
除了 deb 安装器等不考虑像其他发行版移植的应用，不要在 CMakeList 里使用 dpkg 系列命令。

- https://github.com/linuxdeepin/dde-dock/pull/681

## CMAKE_L_FLAGS

[部分项目](https://github.com/linuxdeepin/deepin-movie-reborn/blob/cc35eb1b40214f6beb4c4b21768983e4f0d8d99f/tests/deepin-movie/CMakeLists.txt#L101) 使用了 CMAKE_L_FLAGS，这不是 cmake 标准的变量，这是想用 CMAKE_EXE_LINKER_FLAGS ？

## glib-compile-schemas

gsettings schemas 安装后需要编译一下（一般目录是 /usr/share/glib-2.0/schemas）才能使用，这个应该是安装后进行的，用对应目录所有 gschema.xml 再编译出一个 gschemas.compiled 文件。

像下面这样在 CMakeList.txt 编译是不需要的：
`install(CODE "execute_process(COMMAND glib-compile-schemas ${CMAKE_INSTALL_PREFIX}/share/glib-2.0/schemas)")`
[相关修改](https://github.com/linuxdeepin/dde-session-shell/commit/6faf19b4d73cc35f5cd0f20141077139eccc5846)

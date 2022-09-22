---
title: "编译参数"
date: 2022-09-16T14:00:00+08:00
draft: false
authors: [ "rewine" ]
tags: ["文档"]
---

## 编译参数

CMake 中有许多关于编译器和链接器的设置。当你需要添加一些特殊的需求，你应该首先检查 CMake 是否支持这个需求，如果支持的话，你就可以不用关系编译器的版本，一切交给 CMake 来做即可。 更好的是，你可以在 `CMakeLists.txt` 表明你的意图，而不是通过开启一系列标志 (flag) 。

- CMAKE_BUILD_TYPE

包括 `Debug`, `Release`, `RelWithDebInfo` 和 `MinSizeRe` 4 个选项



全局（或非debug模式）设置 -O3：

CMAKE_BUILD_TYPE  默认 Release

- https://github.com/ros-infrastructure/bloom/issues/327



##  Little libraries

如果你需要链接到 `dl` 库，在 Linux 上可以使用 `-ldl` 标志，不过在 CMake 中只需要在 `target_link_libraries` 命令中使用内置的 CMake 变量 [`${CMAKE_DL_LIBS}` ](https://cmake.org/cmake/help/latest/variable/CMAKE_DL_LIBS.html)。这里不需要模组或者使用 `find_package` 来寻找它。（这个命令包含了调用 `dlopen` 与 `dlclose` 的一切依赖）

## 程序间优化(Interprocedural optimization)

[`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html)，最有名的是 *链接时间优化* 以及 `-flto` 标志，这在最新的几个 CMake 版本中可用。你可以通过变量 [`CMAKE_INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest//CMAKE_INTERPROCEDURAL_OPTIMIZATION.html)（ CMake 3.9+ 可用）或对目标指定 [`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html) 属性来打开它。在 CMake 3.8 中添加了对 GCC 及 Clang 的支持。如果你设置了 `cmake_minimum_required(VERSION 3.9)` 或者更高的版本（参考 [CMP0069](https://cmake.org/cmake/help/latest/policy/CMP0069.html)），当在编译器不支持 [`INTERPROCEDURAL_OPTIMIZATION`](https://cmake.org/cmake/help/latest/prop_tgt/INTERPROCEDURAL_OPTIMIZATION.html) 时，通过变量或属性启用该优化会产生报错。你可以使用内置模块 [CheckIPOSupported](https://cmake.org/cmake/help/latest/module/CheckIPOSupported.html) 中的 `check_ipo_supported()` 来检查编译器是否支持 IPO 。下面是基于 CMake 3.9 的一个例子：

```cmake
include(CheckIPOSupported)
check_ipo_supported(RESULT result)
if(result)
  set_target_properties(foo PROPERTIES INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()
```



## -lpthread

POSIX thread  是基于 C/C++ 的标准线程库。在 cmake 3.1 以上的版本提供了 FindThreads 模块，



```cmake
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(my_app PRIVATE Threads::Threads)
```

https://cmake.org/cmake/help/latest/module/FindThreads.html

https://stackoverflow.com/questions/1620918/cmake-and-libpthread/29871891

参考文档：

https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/chapters/features/small.html



## -std=c++11



## 不要调用 dpkg

## install(CODE "execute_process(COMMAND glib-compile-schemas ${CMAKE_INSTALL_PREFIX}/share/glib-2.0/schemas)")


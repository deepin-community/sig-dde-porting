---
title: "DDE Qt 6.8 适配说明"
date: 2024-10-25
draft: false
authors: [ "BLumia", "tsic404" ]
tags: [ "成果展示" ]
---

# DDE Qt 6.8 适配说明

Qt 6.8 发布已经有一段时间了，各个发行版尝试移植 DDE 时发现包括 dde-shell 在内的几个组件存在比较明显的问题，DDE 小组进行了相关的紧急修复。由于 DDE 部分项目也在分叉维护的状态，为了方便各位移植人员有效进行移植，故在此罗列相关注意事项。

*注：笔者所测试的环境为 Arch Linux，下述为 2024/10/25 testing 仓库状态下的测试结论。若未另行说明，则下述涉及到的项目名称仍然使用了与 DDE 对应项目原始仓库的名称，而非各个发行版下的包名。*

## 分支与 tag 说明

因维护需要，对于部分 DDE 组件（dde-shell、dde-launchpad、dde-tray-loader），我们对 deepin 23 所使用的分支创建了名为 `release/beige` 的维护分支。也会在维护分支上打对应的维护更新用的 tag。

由于 deepin 现阶段的提测流程需要对提测版本打 tag，故我们对主干（master）分支也会打 tag。为了在不与现行规范冲突的情况下尽可能表示区分，我们使用格式为 `x.99.z` 的 tag 标记此版本是尚在开发中的版本。开发中的 tag 版本事实上在满足一定条件下也可供外部使用，但我们不保证 `x.99.z` 中 z 位更新时的兼容性，故仍然建议优先使用 release/beige 上的
tag 版本。

## Qt 6 Wayland

由于 dde-shell 的托盘加载部分（dde-tray-loader）使用了 Wayland（即便是 x11 环境也如此）实现应用的嵌入，故对 Qt 6 的 wayland 组件存在依赖。有下述两个 Patch 需要应用到 Qt 6 Wayland 组件之上：

- <https://codereview.qt-project.org/c/qt/qtwayland/+/598596>
- <https://codereview.qt-project.org/c/qt/qtwayland/+/599732>

## dde-shell

### Patch 说明

升级至 Qt 6.8 后，dde-shell 可能存在面板无任何内容的情况，就于此问题，需要应用这个 patch：

https://github.com/linuxdeepin/dde-shell/commit/46871c83cf8ecfcf83bf2fb49e1f09af997eca96


### 版本建议

- 若目标发行版原本在使用 `1.0.0` 版本，则建议至少更新到 `1.0.2`
- `1.0.3` 以上版本依赖 `treeland-protocols` 项目，进行打包即可，建议对齐打包后更新 dde-shell 至 `1.0.4`
  - 由于 `treeland-protocols` 更新了其 CMake 支持中目标名称的大小写，故你需要打 这个小 patch。
  https://github.com/linuxdeepin/dde-shell/commit/b3f342c094354e4ba87ac1da4cf1a380556b2a3b
- `dde-shell` 主干分支存在 `1.99.1`，但包括此版本在内的主干分支已不再在任务栏提供启动器图标，故需要配合启动器主干分支使用（启动器暂无 `1.99.z` 版本）

tl;dr：建议打包 `treeland-protocols` 后更新至 `1.0.4`。

## dde-tray-loader

### Patch 说明

任务栏托盘区域的弹出面板（例如点击时间组件后的面板）可能有位置不正确的问题，需要应用这个 patch： <https://github.com/linuxdeepin/dde-tray-loader/commit/664b093b6a913764fedbac9110927f26978aa8c9>

### 版本建议

建议更新至 `1.0.4`。

## dde-launchpad

### Patch 说明

启动器的维护分支版本应该可以在无任何修改的情况下正常工作。尽管启动器小窗口模式的面板位置可能不对，但位置问题暂不计划在维护分支解决。

启动器主干分支不存在上述问题，但主干分支暂无 `1.99.z` tag。

### 版本建议

建议更新至 `1.0.5`。

## dde-application-manager

### Patch 说明

不需要 patch。

### 版本建议

一个 deepin 23 的所谓“特性”即，父进程启动的子进程一般会被识别归属为父进程，会导致例如在终端启动 vscode，打开的 vscode 窗口会和终端共用相同图标的问题。此问题已经在最新维护版本得到解决。直接更新dde-shell （>= 1.0.4） dde-application-manager(>=1.2.16)版本即刻解决。

建议更新至 `1.2.16`。

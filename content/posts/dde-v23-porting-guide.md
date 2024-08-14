---
title: "DDE v23 正式版移植简要指南"
date: 2023-08-14
draft: false
authors: [ "BLumia" "rewine" ]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

DDE v23 首个正式版即将随 deepin v23 发布（你阅读到这个文章的时候可能已经发布了）。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。
{.note}
> 

## 概览

即便本次从版本号字面来看可能并没有较大变动，但事实上，本次相比 beta3 -> rc 而言仍然是存在比较大的变化的。本次中，dde-shell 加载托盘插件的策略做了大幅调整，转变为通过 dde-tray-loader 加载插件，托盘区域的插件也放弃了原有的插件，转而移植并使用了来自原 UOS 20 专业版的托盘插件。此外，为了为后续的应用权限管控做准备，本次也对包括 dde-launchpad、dde-shell 等在内的项目调整了其 [DSG 配置文件](https://github.com/linuxdeepin/deepin-specifications/blob/master/unstable/%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%A7%84%E8%8C%83.md) 所使用的应用 ID（`DSG_APP_ID`），故对于移植到其它发行版的情况，若存在相应的 DConfig OEM 配置则也需进行调整。另外，为了解决一些已知的开源合规问题，我们也将原本位于 dtkcore 中的日志部分分离为了一个单独的组件，名为 dtklog。

由于这些项目的版本间互相影响，我们强烈建议移植人员参照 deepin v23 正式版所使用的包版本进行打包（也务必遵循依赖顺序打包）。下面会对主要的部分进行详细说明。

需要注意的是，由于此文章编写时间早于版本发布时间，故最终版本镜像中使用的版本可能高于下面列出的版本。我们尽可能确保此文章的准确性，但若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYSTEM.MANIFEST` (也可能是 `LIVE/FILESYS0.MAN`）路径对应的文件的内容。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于 RC 的版本参照如下：

| package | version |
| --- | --- |
| dtkcommon | 5.6.32 |
| dtklog | 0.0.1 |
| dtkcore | 5.6.32 |
| dtkgui | 5.6.32 |
| dtkwidget | 5.6.32 |
| dtkdeclarative | 5.6.32 |
| qt5integration | 5.6.32 |
| qt5platform-plugins | 5.6.32 |
| dtk6log | 0.0.1 |
| dtk6core | 6.0.18 |
| dtk6gui | 6.0.18 |
| dtk6widget | 6.0.18 |
| dtk6declarative | 6.0.18 |
| qt6integration | 6.0.18 |
| qt6platform-plugins | 6.0.18 |

除新增的 dtklog 外，本次 DTK 版本号以及相对应的平台插件等版本号均已对齐，可直接参照打包。

deepin-kwin wayland 功能已经废弃，未来将由 treeland 替代。目前 dwayland 包已经不再使用，依赖此包的应用比如 qt5platform-plugins， 不应该继续编译依赖 dwayland 的功能，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避。

dtk6 曾对 Qt 6.2, Qt 6.4 和 Qt 6.6 均进行过适配，我们目前的研发与测试主要使用 Qt 6.6 版本，但当前主干也包含了对 Qt 6.7 的支持。如仍发现有 Qt 版本支持问题可在 DDE 移植群（地址见文末）反馈。

目前，使用 dtk6 的正式组件有 dde-application-manager，dde-launchpad 与 dde-shell， 需要注意 dde-shell 的托盘组件 dde-tray-loader 仍然需要使用 qt5。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

下面涉及到的组件的版本参照如下：

| package | version |
| --- | --- |
| deepin-osconfig | 2024.08.06 |
| dde-app-services | 1.0.25 |
| dde-session | 1.2.13 |
| dde-application-manager | 1.2.15 |
| dde-tray-loader | 0.0.8 |
| dde-shell | 0.0.40 |
| dde-launchpad | 0.8.4 |
| dde-application-wizard | 0.1.10 |
| deepin-wayland-protocols | 1.10.0.28 |
| deepin-kwin | 5.27.2.206 |
| dde-launcher | 被 dde-launchpad 取代，不再使用 |
| dde-dock | 被 dde-shell 取代，不再使用 |

### deepin-osconfig

此仓库存放了适用于 deepin 发行版的“OEM”配置。此配置一般 **不需要被其它发行版原样移植**，但考虑到本次存在针对 DSG 应用 ID 的变化调整，故将其列在此处，仅供移植人员参考对应的修改内容，以便当自己所移植到的目标发行版存在 OEM 配置时做出相应的调整。

### dde-application-manager

负责启动和管理 DDE 桌面环境存在的应用程序列表的组件。此组件在 rc 阶段无太大变化（但仍总是建议升级）。

### dde-tray-loader

这是一个新增的组件，旨在提供 DDE 的任务栏托盘部分的各个托盘插件。此项目提供了 xembed、SNI 托盘的加载支持，以及旧式 dde-dock 托盘插件的加载支持（因为并非直接兼容的旧式插件，故此项目同时也提供了这些插件）。

此项目需要配合新的 dde-shell 一同使用。

### dde-shell

dde-shell 旨在将 DDE 桌面环境插件化与模块化，降低开发难度，使各个组件的替换变得更加容易，并且提供更好的桌面环境集成支持。

相较于 RC 阶段，此组件的变化主要在于任务栏托盘区域的加载逻辑的大幅调整与相关动画的调整，以及 shell 本体功能上的支撑调优。对于托盘区域，请参见新增项目 dde-tray-loader 的描述。另外，RC 阶段内附的一些第三方库（例如 networkmanager-qt）已被移除，故一般无需刻意留意是否存在 vendor libs 的问题。

### dde-launchpad

dde-launchpad 相较于 RC 阶段变化并不大，调整大多与缺陷修复以及动画调整有关。需要注意的是，此应用的 DConfig DSG 应用 ID 有变化，故对于移植人员，若原有主动提供针对启动器的 OEM 配置，则需注意修改配置文件放置的对应路径的变化（对应文档也已更新）。

### deepin-kwin

启动器与 shell 均依赖窗管提供一些支持（例如窗口组件的状态、位于的工作区以及层级关系的控制等），故请同时确保 deepin-kwin 使用的版本。

需要注意的是，截至编写此文档时，GitHub 中 deepin-kwin 仓库的最新 tag 落后于上述所列的 deepin-kwin 版本，我们正在与相关项目组协商，若您阅读至此时仍无法在 GitHub 获取到对应的版本且 GitHub 所获取的最新版本存在较大问题，则请暂时先从 deepin v23 (beige) 的软件仓库获取对应源码。

### dde-session

此项目在 RC 阶段无缺陷修复之外的较大变动。

仍需重申，我们已在 beta3 阶段放弃了对 deepin-kwin wayland 的支持，DDE 后续所有 wayland 相关的支持均由 treeland 提供。

## 技术预览组件

为全力确保正式版的版本发布，原本涉及的技术预览组件在 RC 至 release 的这个阶段均无较大进展，故不再于此罗列。

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：[https://t.me/ddeport](https://t.me/ddeport)

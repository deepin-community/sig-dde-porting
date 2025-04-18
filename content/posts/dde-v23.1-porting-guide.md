---
title: "deepin 23.1 DDE 移植简要指南"
date: 2025-04-16
draft: false
authors: [ "BLumia", "rewine" ]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

deepin 23.1 现已发布。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。
{.note}

## 概览

对于 DDE 本次更新并未包含大规模的结构调整，而是比较存催的缺陷修复为主的更新，对于比较值得注意的事项将会列在下方。对于 deepin 23 的注意事项，可参见 deepin 23 正式版发布时的移植指南文章所给出的说明。

由于 DDE 涉及到的各个组件项目的版本间互相影响，我们强烈建议移植人员参照 deepin 23.1 正式版所使用的包版本进行打包（也务必遵循依赖顺序打包）。下面会对主要的部分进行详细说明。

下面给出的版本号信息供打包移植时参考。若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYSTEM.MANIFEST` (也可能是 `LIVE/FILESYS0.MAN`）路径对应的文件的内容。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于 RC 的版本参照如下：

| package | version |
| --- | --- |
| dtkcommon | 5.7.5 |
| dtklog | 0.0.2 |
| dtkcore | 5.7.5 |
| dtkgui | 5.7.5 |
| dtkwidget | 5.7.5 |
| dtkdeclarative | 5.7.5 |
| qt5integration | 5.7.5 |
| qt5platform-plugins | 5.7.5 |
| dtk6log | 0.0.2 |
| dtk6core | 6.0.25 |
| dtk6gui | 6.0.25 |
| dtk6widget | 6.0.25 |
| dtk6declarative | 6.0.25 |
| qt6integration | 6.0.25 |
| qt6platform-plugins | 6.0.25 |

除新增的 dtklog 外，本次 DTK 版本号以及相对应的平台插件等版本号均已对齐，可直接参照打包。

deepin-kwin wayland 功能已经废弃，未来将由 treeland 替代。目前 dwayland 包已经不再使用，依赖此包的应用比如 qt5platform-plugins，不应该继续编译依赖 dwayland 的功能，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避。

目前，使用 dtk6 的正式组件有 dde-application-manager，dde-launchpad 与 dde-shell。需要注意，deepin 23 环境中 dde-shell 的托盘组件 dde-tray-loader 仍然需要使用 qt5。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

下面涉及到的组件的版本参照如下：

| package | version |
| --- | --- |
| deepin-osconfig | 2024.08.06 |
| dde-app-services | 1.0.27 |
| dde-session | 1.2.13 |
| dde-application-manager | 1.2.27 |
| dde-tray-loader | 1.0.10 |
| dde-shell | 1.0.10 |
| dde-launchpad | 1.0.11 |
| dde-application-wizard | 0.1.10 |
| deepin-wayland-protocols | 1.10.0.28 |
| deepin-kwin | 5.27.2.213 |
| dde-launcher | 被 dde-launchpad 取代，不再使用 |
| dde-dock | 被 dde-shell 取代，不再使用 |

### dde-application-manager

此组件现已使用主干分支最新版本（当前为 `1.2.27`）。请注意，较早的主干版本（例如 `1.2.26`）在 deepin 23 环境存在一些已知行为问题，故移植最新的 deepin 23 DDE 时，请至少使用 `1.2.27` 版本。

### dde-session-shell

尽管此组件不存在架构性质层面的较大调整，但涉及到打包移植相关的注意事项。此组件由于主干分支的研发需求，对仓库进行过迁移到。当前 GitHub 上的 linuxdeepin/dde-session-shell 仓库历史已与之前不同。故如果你需要适用于 deepin 23 的此仓库的完整历史，请转到 [dde-session-shell-snipe](https://github.com/linuxdeepin/dde-session-shell-snipe)。所有原始仓库的提交历史以及 tag 均可在这个仓库中找到（实质是仓库重命名后新建了与原名的同名仓库）。

（注：相关请参见[此邮件列表存档](https://www.freelists.org/post/deepin-devel/githubddesessionshellddesessionshell)）

## Qt 6.9 编译问题
- dtk：23 版本 dtk 没有适配 qt 6.9,可尝试使用最新 tag,或者参考 Arch linux 打包提供的 patch
- dde-shell：需要 https://github.com/linuxdeepin/dde-shell/pull/1091 

如果遇到 qmlsc 崩溃问题，见 [QTBUG-135885](https://bugreports.qt.io/browse/QTBUG-135885) 和 [QTBUG-135885](https://bugreports.qt.io/browse/QTBUG-135288)，需要为 qtdeclarative 增加以下 patch：

- https://github.com/qt/qtdeclarative/commit/d1aa2e8466bab73c3e4d120356238b482b55f02a.patch


## 技术预览组件

原本涉及的技术预览组件在 23 至 23.1 的这个阶段均无较大进展，故不再于此罗列。

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：[https://t.me/ddeport](https://t.me/ddeport)

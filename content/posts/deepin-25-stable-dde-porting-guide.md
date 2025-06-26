---
title: "deepin 25 正式版 DDE 移植简要指南"
date: 2025-06-25
draft: false
authors: [ "BLumia" ]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

即将发布（你阅读到这个文章的时候可能已经发布了）的 deepin 25 正式版将会包含对应的新版 DDE。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。

## 概览

相对于 deepin 25 beta，在 deepin 25 正式版中并不存在较大幅的架构调整，而是以缺陷修复以及完善之前尚未完善但计划涵盖在最终版本的组件（例如 QML 版控制中心）作为研发的重心。本阶段中有部分组件的首位版本号存在调整，另存在一些注意事项，详情请参见后续的描述。

由于这些项目的版本间互相影响，我们建议移植人员参照 deepin 25 beta 所使用的包版本进行打包，下面会对主要的部分进行详细说明。

需要注意的是，由于此文章编写时间早于版本发布时间，故最终版本镜像中使用的版本可能高于下面列出的版本。我们尽可能确保此文章的准确性，但若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYS{T,0,1}.MAN`/`live/filesystem.manifest` 路径对应的文件的内容。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于deepin 25 正式版的版本参照如下：

package         | version
----------------|--------
dtkcommon       | 5.7.17
dtklog         | 0.0.4
dtkcore         | 5.7.17
dtkgui          | 5.7.17
dtkwidget       | 5.7.17
dtkdeclarative  | 5.7.17
qt5integration  | 5.7.17
qt5platform-plugins | 5.7.17.1
dtk6core        | 6.0.37
dtk6gui         | 6.0.37
dtk6widget      | 6.0.37
dtk6declarative | 6.0.37
qt6integration  | 6.0.37
qt6platform-plugins | 6.0.37

本次 DTK 组件大部分版本号以及相对应的平台插件等版本号均已对齐，例外的有 dtkcommon 与 dtklog。可参照上表进行打包。

关于 qt5platform-plugins，现有的 dwayland 插件可能对非 DDE 环境（例如 KDE）的 wayland 用户存在影响，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避影响。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

在 alpha 与 beta 阶段使用 1.99.z 系列版本号的项目现均启用了 2.0.z 版本号，以表示相关的行为变动。针对 dde-shell 所提供库的依赖的 SOVERSION 仍为 1，不受打包版本号的影响。

下面涉及到的组件的版本参照如下：

package        | version
---------------|--------
deepin-desktop-schemas | 6.0.11
dde-daemon | 6.1.40
dde-session    | 2.0.2
dde-session-ui | 6.0.30
dde-session-shell | 6.0.41
dde-application-manager | 1.2.31
dde-shell      | 2.0.1
dde-launchpad  | 2.0.1
dde-tray-loader | 2.0.2
dde-application-wizard | 0.1.15
dde-clipboard | 6.1.9
deepin-service-manager | 1.0.13
dde-launcher | 被 dde-launchpad 取代，不再使用
dde-dock       | 被 dde-shell 取代，不再使用
startdde        | 已被废弃，不再使用

#### dde-application-manager

由于涉及到诸多关于应用识别的改善，故建议总是使用最新版本。

#### dde-session-shell

相比 beta ，此项目不涉及较大的结构调整，但你可能希望阅读 beta 时的变动描述，涉及到了仓库位置和提交历史相关的说明。

#### dde-shell

dde-shell 旨在将 DDE 桌面环境插件化与模块化，降低开发难度，使各个组件的替换变得更加容易，并且提供更好的桌面环境集成支持。正式阶段相比 beta 阶段集中在缺陷的修复上，并未涵盖太多的结构调整和新特性。对于 beta 以及更早版本的变化，请阅读之前的博客文章。

为保障 dde-shell 在 Qt 6.8.0 或 6.8.1 的环境可以正常运行（即使是X11环境下），若 ，则 **必须** 给 qtwayland 打下面的 patch：

- https://codereview.qt-project.org/c/qt/qtwayland/+/603556 （[此 patch 已合入 Qt 上游](https://github.com/qt/qtwayland/commit/070414dd4155e13583e5e8b16bed1a5b68d32910)，涵盖在 `Qt Wayland >= 6.8.2` 版本中）

dde-shell 在 alpha 中为修正一个特定问题所包含的一个变更依赖另一个 Qt Wayland 的 patch：

- https://github.com/deepin-community/qt6-wayland/pull/12

若你所移植的目标发行版不接受此补丁，则可考虑对 dde-shell 项目 revert 于此相关的对应 commit：

- https://github.com/linuxdeepin/dde-shell/commit/77cac3d5a346728f6678eedc77fce404f0ffd8b6

#### dde-launchpad

dde-launchpad 现仅支持以 dde-shell 插件的形式被最终用户使用。因而，打包 dde-launchpad 现需要先打包 dde-shell，并确保用户最终使用的是 dde-shell。

#### dde-session

我们已在 deepin 23 beta3 起放弃了对 deepin-kwin wayland 的支持，DDE 后续所有 wayland 相关的支持均由 treeland 提供。请参见后续的 Treeland 段落。

下面涉及到的组件的版本参照如下。对于位于非 linuxdeepin 组织的软件包，此处一并给出了组织名：

package        | version
---------------|--------
vioken/waylib | 0.6.13
vioken/qwlroots | 0.5.3
treeland | 0.5.20
ddm | 0.1.10

## deepin-kwin 环境

此部分相对 beta 也未存在较大变动，但由于此组件的重要性，方便起见，此处重新阐述 beta 阶段的变化：

deepin-kwin 对 Qt 的版本依赖切换到了 Qt 6，不过值得注意的是，当前的 deepin-kwin 并非从上游 kwin 6.x 中 fork 出来的，而是基于 uos 20 版本的 kwin 5.27.x 进行的 qt6/kf6 迁移，并且由于一些原因，其二进制可执行的文件名恢复到了 kwin，这会导致与上游原版 kwin 的冲突。

就于此事项的详细介绍，请参见 https://github.com/orgs/linuxdeepin/discussions/11471 讨论。对于移植人员，我们建议考虑下面三种方案：

- 使用 Qt 6 的 deepin-kwin，放弃与上游 kwin 的共存支持。
- 继续使用 deepin-kwin 5.27，尽管存在一些问题，但此版本仍然可保证和上游版本 kwin 共存。
- 考虑打包 Treeland 环境。

## Treeland 环境

Treeland 环境在 alpha -> beta 阶段无较大变化，alpha 阶段的移植注意事项仍然适用于 beta。

### DDM

尽管 DDM 目前是基本功能可用状态，DDM 目前仍相对而言不够稳定。对于打包移植而言，建议采用其他DM来启动用户级的treeland。

对于其它 DM，只需要打包时安装 `usr/share/wayland-sessions/treeland-user.desktop` 即可。

### Qt 补丁

下述假定您的发行版使用的 Qt 版本为 Qt 6.8.2。

如果你在 Treeland 下遇到小 launchpad 无法输入中文的问题，可以打下面的 patch，但是该 patch 目前尚未进行完整测试，可能存在一些问题。

https://codereview.qt-project.org/c/qt/qtbase/+/611940

另外，如果你的发行版所附的 Qt 6.8 版本并未更新至 Qt 6.8.2，则可能需要打三个额外的补丁，可参见 [DDE Qt 6.8 适配说明（针对 Qt 6.8.0）](https://deepin-community.github.io/sig-dde-porting/posts/dde-qt6.8-porting-guide/) 以及 [deepin 25 preview DDE 移植简要指南（针对 Qt 6.8.1）](https://deepin-community.github.io/sig-dde-porting/posts/deepin-25-preview-dde-porting-guide/#qt-%E8%A1%A5%E4%B8%81)。

#### Qt 6.9 说明

DDE 尚未完整针对 Qt 6.9 进行测试，但存在一些已知问题，下面列出了对应的问题和处理方式：

1. QML 的 DelegateChoice/DelegateChooser 组件不存在问题

Qt 6.9 中，此组件被从 labs “转正”到了 `QtQml.Models` 下，相应的 import 需要修正。已知 dde-shell 存在此问题（[修正相关问题的 patch](https://github.com/linuxdeepin/dde-shell/pull/1173)）。

若有其他项目存在相同问题，则对于此问题的修正，可参考 dde-shell 的相关提交：

- https://github.com/linuxdeepin/dde-shell/pull/1173

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：<https://t.me/ddeport>

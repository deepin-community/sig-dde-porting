---
title: "deepin 25 preview DDE 移植简要指南"
date: 2025-01-16
draft: false
authors: [ "BLumia", "tsic404" ]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

即将发布（你阅读到这个文章的时候可能已经发布了）的 deepin 25 preview 将会包含对应的新版 DDE。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。
{.note}

## 概览

相对于 deepin 23，在 deepin 25 中，包括桌面、通知中心在内的大部分旧的 DDE 桌面组件已转化为 dde-shell 插件形式，以供更好的跨显示环境兼容性。包括控制中心在内的组件也已开始提供全新设计以及基于 QML 的全新界面。同时，我们也对 Qt、DTK 进行了更多完善，以供 DDE 组件以及 Treeland 能够更好的运行。

由于这些项目的版本间互相影响，我们建议移植人员参照 deepin 25 preview 所使用的包版本进行打包，下面会对主要的部分进行详细说明。

需要注意的是，由于此文章编写时间早于版本发布时间，故最终版本镜像中使用的版本可能高于下面列出的版本。我们尽可能确保此文章的准确性，但若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYS{0,1}.MAN`/`live/filesystem.manifest` 路径对应的文件的内容。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于 RC 的版本参照如下：

package         | version
----------------|--------
dtkcommon       | 5.7.7
dtklog         | 0.0.2
dtkcore         | 5.7.7
dtkgui          | 5.7.7
dtkwidget       | 5.7.7
dtkdeclarative  | 5.7.7
qt5integration  | 5.7.7
qt5platform-plugins | 5.7.7
dtk6core        | 6.0.27
dtk6gui         | 6.0.27
dtk6widget      | 6.0.27
dtk6declarative | 6.0.27
qt6integration  | 6.0.27
qt6platform-plugins | 6.0.27

除 dtklog 外，本次 DTK 版本号以及相对应的平台插件等版本号均已对齐，可直接参照打包。

关于 qt5platform-plugins，现有的 dwayland 插件可能对非 DDE 环境（例如 KDE）的 wayland 用户存在影响，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避影响。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

由于 deepin 25 preview 仍在持续开发过程中，故较多组件采取了 x.99.z 的版本号策略。此外，一般情况下，此类 tag 并不会实际以 git tag 的形式存在，而只会体现在 `debian/changlog` 文件中。下面涉及到的此类版本号将会在版本发布前后补充对应的 git tag。

下面涉及到的组件的版本参照如下：

package        | version
---------------|--------
dde-session    | 1.99.7
dde-application-manager | 1.2.23
dde-shell      | 1.99.19
dde-launchpad  | 1.99.5
dde-tray-loader | 1.99.12
dde-application-wizard | 0.1.11
dde-clipboard | 6.1.4
dde-launcher | 被 dde-launchpad 取代，不再使用
dde-dock       | 被 dde-shell 取代，不再使用

#### dde-application-manager

由于涉及到诸多关于应用识别的改善，故建议总是使用最新版本。

#### dde-shell

dde-shell 旨在将 DDE 桌面环境插件化与模块化，降低开发难度，使各个组件的替换变得更加容易，并且提供更好的桌面环境集成支持。preview 阶段，dde-shell 已经可以满足原计划的部分目标。现 DDE 环境下，dde-shell 已取代 dde-dock 来负责管理整个 dock 区域、 dde-launchpad 提供了对应的 dde-shell 插件用以展示启动器相关的界面、原 dde-session-ui 中的通知中心部分也转到了 dde-shell 中，且转用了新的界面设计。

关于 shell 的服务启动方面，为了方便故障排查，dde-shell 从原本的单进程转为了两个进程（分别提供桌面和任务栏两个部分）。另外，shell项目的任务栏部分在此阶段也配合 dde-application-manager 对应用识别的准确度进行了诸多完善。若仍有发现应用错误识别和错误分组的问题，欢迎及时反馈。

为保障dde-shell在Qt6.8之后的环境可以正常运行（即使是X11环境下），**必须**给qtwayland打下面的patch：

https://codereview.qt-project.org/c/qt/qtwayland/+/603556

#### dde-launchpad

dde-launchpad 现仅支持以 dde-shell 插件的形式被最终用户使用。因而，打包 dde-launchpad 现需要先打包 dde-shell，并确保用户最终使用的是 dde-shell。

#### dde-session

需要注意的是，我们已在 deepin 23 beta3 起放弃了对 deepin-kwin wayland 的支持，DDE 后续所有 wayland 相关的支持均由 treeland 提供。请参见后续的 Treeland 段落。

下面涉及到的组件的版本参照如下。对于位于非 linuxdeepin 组织的软件包，此处一并给出了组织名：

package        | version
---------------|--------
vioken/waylib | 0.6.10
vioken/qwlroots | 0.5.2
treeland | 0.5.17
ddm | 0.1.9


## Treeland 环境

Treeland 环境相较于 deepin 23 阶段有了较多的提升，不过由于 Treeland 迭代开发过程中我们对 Qt 以及 wlroots 进行了诸多完善，故 Treeland 对 Qt 以及 wlroots 等组件有较高的版本要求，以及可能需要应用一些额外的 patch。

### DDM

尽管 DDM 目前是基本功能可用状态，DDM 目前仍相对而言不够稳定。对于打包移植而言，建议采用其他DM来启动用户级的treeland。

对于其它 DM，只需要打包时安装 `usr/share/wayland-sessions/treeland-user.desktop` 即可。

### Qt 补丁

下述假定您的发行版使用的 Qt 版本为 Qt 6.8.1。

为保障 dde-shell 在 Treeland 上可以正常运行，需要打下面的 patch，否则可能会出现 dde-shell 崩溃的情况。

https://codereview.qt-project.org/c/qt/qtbase/+/607654

如果你在 Treeland 下遇到小 launchpad 无法输入中文的问题，可以打下面的 patch，但是该 patch 目前尚未进行完整测试，可能存在一些问题。

https://codereview.qt-project.org/c/qt/qtbase/+/611940

另外，如果你的发行版所附的 Qt 6.8 版本并未更新至 Qt 6.8.1，则可能需要打两个额外的补丁，可参见 [DDE Qt 6.8 适配说明](https://deepin-community.github.io/sig-dde-porting/posts/dde-qt6.8-porting-guide/)。

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：<https://t.me/ddeport>

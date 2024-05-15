---
title: "DDE v23 RC 移植简要指南"
date: 2023-05-14
draft: false
authors: [ "BLumia", "tsic404" ]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

DDE v23 RC 即将随 deepin v23 RC 发布（你阅读到这个文章的时候可能已经发布了）。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。
{.note}

## 概览

相比 beta2 -> beta3 而言，原本处于技术预览状态的 dde-shell 项目现已开始逐步取代部分旧的 DDE 组件。DDE 此次 beta3 -> RC 的更新中，dde-shell 取代了 dde-dock 项目，dde-launchpad 也开始转为使用 dde-shell 的对应插件版本。同时，由于对 Qt6 与 DTK6 使用的增加，我们也对 DTK 进行了大量的问题修复，这些修复也被对应的组件（dde-launchpad 与 dde-shell）依赖。

由于这些项目的版本间互相影响，我们建议移植人员参照 deepin v23 RC 所使用的包版本进行打包，下面会对主要的部分进行详细说明。

需要注意的是，由于此文章编写时间早于版本发布时间，故最终版本镜像中使用的版本可能高于下面列出的版本。我们尽可能确保此文章的准确性，但若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYS0.MAN` 路径对应的文件的内容。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于 RC 的版本参照如下：

package         | version
----------------|--------
dtkcommon       | 5.6.29
dtkcore         | 5.6.29
dtkgui          | 5.6.29
dtkwidget       | 5.6.29
dtkdeclarative  | 5.6.29
qt5integration  | 5.6.29
qt5platform-plugins | 5.6.29
dtk6core        | 6.0.16
dtk6gui         | 6.0.16
dtk6widget      | 6.0.16
dtk6declarative | 6.0.16
qt6integration  | 6.0.16
qt6platform-plugins | 6.0.16

本次 DTK 版本号以及相对应的平台插件等版本号均已对齐，可直接参照打包。

关于 qt5platform-plugins，现有的 dwayland 插件可能对非 DDE 环境（例如 KDE）的 wayland 用户存在影响，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避影响。

dtk6 曾对 Qt 6.2, Qt 6.4 和 Qt 6.6 均进行过适配，我们目前的研发与测试主要使用 Qt 6.6 版本。对于 Qt 6.7 我们暂未进行主动支持，对于 Qt 6.7 的相关处理请先参照 [Arch Linux 目前使用的 `qt-6.7.patch`](https://gitlab.archlinux.org/archlinux/packaging/packages/deepin-qt6platform-plugins/-/blob/9b6252bc386659bf7f586578dea394be6ff40462/qt-6.7.patch)。

目前，使用 dtk6 的正式组件有 dde-application-manager，dde-launchpad 与 dde-shell，技术预览组件有 treeland。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

下面涉及到的组件的版本参照如下：

package        | version
---------------|--------
dde-session    | 1.2.9
dde-application-manager | 1.2.12
dde-shell      | 0.0.23
dde-launchpad  | 0.6.12
dde-application-wizard | 0.1.5
dde-dock       | 被 dde-shell 取代，不再使用

#### dde-application-manager

本次此组件并不存在如 beta2 -> beta3 阶段时的巨大变化，但由于此组件仍为诸多组件的核心依赖，并且此组件也因其它组件需要而增加了一些新的接口，故建议总是使用最新版本。

#### dde-shell

dde-shell 旨在将 DDE 桌面环境插件化与模块化，降低开发难度，使各个组件的替换变得更加容易，并且提供更好的桌面环境集成支持。RC 阶段，dde-shell 已经可以满足原计划的部分目标。现 DDE 环境下，dde-shell 已取代 dde-dock 来负责管理整个 dock 区域，并且 dde-launchpad 也提供了（且默认使用）对应的 dde-shell 插件，用以展示启动器相关的界面。

尽管 dde-shell 也包含了一份通知中心插件，但这个插件并不会在 RC 中被启用。

出于初期快速开发目的，dde-shell 本身内附了诸多其它项目的托盘插件代码以及一些第三方库，这部分的代码会在后续版本迭代中逐渐被调整或移除。对于这些代码，可在移植过程中先行酌情调整或移除。项目内附的 networkmanager-qt 依赖也会在后续移除，转而使用系统版本。由于 RC 版本不会涵盖此变更，若有需要，请参见 [linuxdeepin/dde-shell#286](https://github.com/linuxdeepin/dde-shell/pull/286)。

#### dde-launchpad

RC 版本中，dde-launchpad 不再支持 Qt 5 构建，也不再直接支持以独立进程的形式被最终用户使用（事实上仍然支持，但仅供开发调试场景下使用）而仅支持以 dde-shell 插件的形式被最终用户使用。因而，打包 dde-launchpad 现需要先打包 dde-shell，并确保用户最终使用的是 dde-shell。

#### dde-session

在之前的版本中，dde-session 提供的相关 systemd 服务依赖关系会导致进入 KDE 环境时也会拉起 DDE 相关的服务，最终由于拉起失败导致 KDE 环境也无法进入。在 RC 版本的 dde-session 中，此问题应当已被解决。

另外重申，我们已在 beta3 阶段放弃了对 deepin-kwin wayland 的支持，DDE 后续所有 wayland 相关的支持均由 treeland 提供。

## 技术预览组件

技术预览阶段的组件均可酌情决定是否打包。需要注意的是，这些组件并不一定在 RC 阶段存在改善。

### treeland / ddm

treeland 目前是技术预览阶段的项目，兼顾 Wayland 窗口合成器和显示管理功能。treeland 除需要 dtk6declarative 外，还依赖新项目包括 [waylib](https://github.com/vioken/waylib) 和 [qwlroots](https://github.com/vioken/qwlroots)。需要使用 Qt 版本 >= 6.6.0， wlroots 版本 >= 0.17.0 编译 qwlroots/waylib。

ddm 的仓库也在 RC 阶段进行了拆分，现在 ddm 与 treeland 已是分别独立的两个仓库进行维护了。

### deepin-im

treeland 目前是技术预览阶段的项目，提供了输入法的抽象层。目前，deepin-im 仅期望在 treeland 环境下使用。它会设置 `QT_IM_MODULE` 等环境变量的值来影响实际使用的输入法模块。

RC 版本中，并未涉及对 deepin-im 组件的改善。

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：<https://t.me/ddeport>


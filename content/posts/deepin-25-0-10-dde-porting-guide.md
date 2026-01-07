---
title: "deepin 25.0.10 正式版 DDE 移植简要指南"
date: 2026-01-07
draft: false
# ↓编辑的话请把自己的名字加到下面的作者名单里↓
authors: [ "BLumia" ]
tags: [ "成果展示" ]
---

deepin 25.0.10 已经发布一段时间了，而即将发布的 deepin 25.0.10 正式版镜像也会包含对应版本的 DDE 组件。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。

## 概览

相对于 deepin 25 最初的正式版本，在 deepin 25.0.10 中并不存在较大幅的架构调整，而是以缺陷修复作为研发的重心，并涵盖了一些小功能的更新。本阶段中有部分组件的首位版本号存在调整，另存在一些注意事项，详情请参见后续的描述。

由于这些项目的版本间互相影响，我们建议移植人员参照 deepin 25 中实际所使用的包版本进行打包，下面会对主要的部分进行详细说明。

需要注意的是，由于此文章编写时间早于新 ISO 的发布时间，故最终版本镜像中使用的版本可能高于下面列出的版本。我们尽可能确保此文章的准确性，但若您需要获取 ISO 镜像中使用的确切软件版本列表，请挂载 ISO 后参阅 `LIVE/FILESYS{T,0,1}.MAN`/`live/filesystem.manifest` 路径对应的文件的内容。

另外，由于前述 manifest 文件包含了 ISO 所附带的所有软件包的版本信息，而 DDE 移植并不需要关注整个列表，你可以参考 [deepin-community/deepin-desktop-environment](https://github.com/deepin-community/deepin-desktop-environment/) 提供的几个 Meta 包中描述的依赖关系，来了解哪些软件包和 DDE 有关。

## 主要组件

### DTK 与 DTK6

DTK 是 DDE 组件与应用的基础依赖，适用于deepin 25 正式版的版本参照如下：

project         | package name[^1] | version
----------------|---------------|--------
dtkcommon       | libdtkdata    | 5.7.26
dtklog          | libdtklog     | 0.0.6
dtkcore         | libdtkcore5   | 5.7.26
dtkgui          | libdtkgui5    | 5.7.26
dtkwidget       | libdtkwidget5 | 5.7.26.1
dtkdeclarative  | libdtkdeclarative5 | 5.7.26
qt5integration  | dde-qt5integration | 5.7.26
qt5platform-plugins | dde-qt5xcb-plugin | 5.7.26
dtk6core        | libdtk6core   | 6.0.46
dtk6gui         | libdtk6gui    | 6.0.46
dtk6widget      | libdtk6widget | 6.0.46.1
dtk6declarative | libdtk6declarative | 6.0.46
qt6integration  | dde-qt6integration | 6.0.46
qt6platform-plugins | dde-qt6xcb-plugin | 6.0.46

[^1]: 一个项目可能有多个软件包，此处的包名仅列出了此项目在 deepin 发行版中具有代表性的包名

本次 DTK 组件大部分版本号以及相对应的平台插件等版本号均已对齐，例外的有 dtkcommon 与 dtklog。可参照上表进行打包。

关于 qt5platform-plugins，现有的 dwayland 插件可能对非 DDE 环境（例如 KDE）的 wayland 用户存在影响，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避影响。

另外，之前版本的打包采取了 dtk5 与 dtk6 分仓库进行打包的方案，就于上述（在 deepin 25.0.10 镜像中使用的 DTK）版本，您暂时仍然需要在 `dtk6blah` 仓库中获取对应的 tag。目前正在向回归到同一仓库打包的方案进行过度，且过度即将完成，即未来版本将会不再需要使用 `dtk6blah` 仓库，只需 `dtkblah` 即可。对于此事项的进展可以关注移植小组的 Telegram 或 Matrix 群聊。

### DDE 主要组件

下面仅涉及变化较大或影响较广的组件。其余未涉及的组件可正常参照最新 tag 进行打包与移植。

在 alpha 与 beta 阶段使用 1.99.z 系列版本号的项目现均启用了 2.0.z 版本号，以表示相关的行为变动。针对 dde-shell 所提供库的依赖的 SOVERSION 仍为 1，不受打包版本号的影响。

下面涉及到的组件的版本参照如下：

project         | package name[^1] | version
----------------|---------------|--------
deepin-desktop-schemas | deepin-desktop-schemas | 6.0.13
dde-daemon     | dde-daemon  | 6.1.64
dde-session    | dde-session | 2.0.9
dde-session-ui | dde-session-ui | 6.0.35
dde-session-shell | dde-session-shell | 6.0.50
dde-application-manager | dde-application-manager | 1.2.37
dde-shell      | dde-shell   | 2.0.19
dde-launchpad  | dde-launchpad | 2.0.17
dde-tray-loader | dde-tray-loader | 2.0.17
dde-application-wizard | dde-application-wizard-daemon-compat | 0.1.21
dde-clipboard | dde-clipboard | 6.1.16
deepin-service-manager | deepin-service-manager | 1.0.17
treeland-protocols | treeland-protocols | 0.5.1
dde-network-core | libdde-network-core | 2.0.74
dde-control-center | dde-control-center | 6.1.58
dde-launcher   | 被 dde-launchpad 取代，不再使用
dde-dock       | 被 dde-shell 取代，不再使用
startdde       | 已被废弃，不再使用

#### dde-application-manager

由于涉及到诸多关于应用识别的改善，故建议总是使用最新版本。

#### dde-session-shell

请注意，当前项目获取 tag 的源码仓库现在变为了 https://github.com/linuxdeepin/dde-session-shell-snipe 。

#### dde-shell

dde-shell 在本段时间的更新迭代中，修复了若干关于任务栏图标识别问题以及图标错位的问题。对于比较受到争议的 cgroup 识别问题也给出了配置项运行用户进行微调。对于移植，也处理了高版本 Qt 的一些兼容性问题。

我们假定您要移植的目标发行版使用的 Qt 版本大于 6.8.2，如果早于此小版本，请阅读上一篇（正式版时的）博客来了解相关的补丁信息。

dde-shell 在 alpha 中为修正一个特定问题所包含的一个变更依赖另一个 Qt Wayland 的 patch：

- https://github.com/deepin-community/qt6-wayland/pull/12

若你所移植的目标发行版不接受此补丁，则可考虑对 dde-shell 项目 revert 于此相关的对应 commit：

- https://github.com/linuxdeepin/dde-shell/commit/77cac3d5a346728f6678eedc77fce404f0ffd8b6

#### dde-launchpad

dde-launchpad 现仅支持以 dde-shell 插件的形式被最终用户使用。因而，打包 dde-launchpad 现需要先打包 dde-shell，并确保用户最终使用的是 dde-shell。

#### dde-session

我们已在 deepin 23 beta3 起放弃了对 deepin-kwin wayland 的支持，DDE 后续所有 wayland 相关的支持均由 treeland 提供。请参见后续的 Treeland 段落。

## deepin-kwin 环境

此部分相对 beta 也未存在较大变动，但由于此组件的重要性，方便起见，此处重新阐述 beta 阶段的变化：

deepin-kwin 对 Qt 的版本依赖切换到了 Qt 6，不过值得注意的是，当前的 deepin-kwin 并非从上游 kwin 6.x 中 fork 出来的，而是基于 uos 20 版本的 kwin 5.27.x 进行的 qt6/kf6 迁移，并且由于一些原因，其二进制可执行的文件名恢复到了 kwin，这会导致与上游原版 kwin 的冲突。

就于此事项的详细介绍，请参见 https://github.com/orgs/linuxdeepin/discussions/11471 讨论。对于移植人员，我们建议考虑下面三种方案：

- 使用 Qt 6 的 deepin-kwin，放弃与上游 kwin 的共存支持。
- 继续使用 deepin-kwin 5.27，尽管存在一些问题，但此版本仍然可保证和上游版本 kwin 共存。
- 考虑打包 Treeland 环境。

## Treeland 环境

Treeland 环境尽管目前仍然是实验性支持，但目前 treeland 构建不再需要事先构建 waylib 与 qwlroots。

下面涉及到的组件的版本参照如下。对于位于非 linuxdeepin 组织的软件包，此处一并给出了组织名：

package        | version
---------------|--------
treeland-protocols | 0.5.1
treeland       | 0.7.8
ddm            | 0.2.3

### DDM

尽管 DDM 目前是基本功能可用状态，DDM 目前仍相对而言不够稳定。对于打包移植而言，建议采用其他DM来启动用户级的treeland。

对于其它 DM，只需要打包时安装 `usr/share/wayland-sessions/treeland-user.desktop` 即可。

### Qt 补丁

我们假定您要移植的目标发行版使用的 Qt 版本大于 6.8.2，如果早于此小版本，请阅读上一篇（正式版时的）博客来了解相关的补丁信息。

如果你在 Treeland 下遇到小 launchpad 无法输入中文的问题，可以打下面的 patch，但是该 patch 目前尚未进行完整测试，可能存在一些问题。

https://codereview.qt-project.org/c/qt/qtbase/+/611940

## 建议忽略移植的组件

### dde-api-proxy

此组件的存在目的是给部分仍在依赖 deepin 20/23 所提供的 D-Bus 服务的第三方组件提供兼容。DDE 和 deepin 第一方应用均不依赖这些旧的 D-Bus 接口，且此组件目前不被 deepin 自身所需，所以此项目不应当被移植。

### deepin-anything

尽管被部分 DDE 组件依赖，但均为可选。anything 提供了内核模块，对于滚动发行版移植可能不友好，可能移植并不能得到很高的体验改善。

若忽略 deepin-anything 移植，则下列组件也应当被忽略（因为依赖了 deepin-anything）：

- dde-grand-search

### dde-application-wizard

尽管此项目初衷之一是提供可移植的模块化卸载服务，但并未达到理想状态。尽管事实上可被顺利移植，此项目可以考虑忽略。缺失此组件并不会影响 DDE 主要功能。

## 已知问题

下述问题是已知问题，需酌情处理。

### deepin-kwin 与 kwin 原版的共存问题

目前由于 Qt 6 版的 deepin-kwin 与上游原版 kwin 存在一些同名可执行文件/配置文件，故无法做到 deepin-kwin 与 kwin 的共存。若要移植 x11 会话，选择如下：

1. 使用 Qt 5 版本的 deepin-kwin（目前 openSUSE 移植的做法如此）
2. 打包 deepin-kwin，并声明与上游 kwin 的冲突（会导致 KDE x11 与 DDE x11 会话无法共存，目前 Arch Linux 移植的做法如此）

由于 dde-session 的依赖关系，这也会导致如果仅希望使用 deepin 的 wayland 会话，也会受到上述问题的影响而无法使 KDE 与 DDE 共存，此问题会在后续进行解决来确保允许仅移植 DDE wayland，从而使 DDE wayland 和 KDE 可以无冲突的共存。

### 任务栏托盘部分，插件的背景颜色异常

这个问题是目前移植过程中发现的已知问题，但暂时没有时间定位分析。若您移植过程中遇到此问题，可暂时忽略，当然也欢迎协助定位并提交 PR :)

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：<https://t.me/ddeport>

---
title: "DDE v23 beta 3 移植简要指南"
date: 2023-02-02
draft: false
authors: [ "BLumia", "rewinee", "justforlxz"]
tags: [ "成果展示" ]
---

*编辑的话请把自己的名字加到作者名单里*

DDE v23 beta3 即将随 deepin v23 beta 3 发布（你阅读到这个文章的时候可能已经发布了）。为了方便各个其它发行版的包维护者可以更方便的移植 DDE 到对应的发行版，这里提供一篇简要的移植指南，用以描述常见的移植问题和解决方案。

> 下面对项目名称的称呼均以 GitHub 对应的原始仓库名为准。
{.note}

## 概览

DDE 此次 beta2 -> beta3 的更新中，dde-application-manager 进行了大规模重构；dde-launchpad 取代 dde-launcher 成为了新的启动器/开始菜单应用；新的技术预览项目 dde-shell 和 treeland 也随此次发布提供了初步版本，以及为了服务 dde-launcher 和 dde-shell，dtk 也开始提供 Qt 6 版本。

由于这些项目的版本间互相影响，我们建议移植人员参照 deepin v23 beta3 所使用的包版本进行打包（随后会把完整的版本参照列表贴到这里），下面会对主要的部分进行详细说明。

## 主要组件

### DTK 与 DTK6

作为 DDE 组件与应用的基础依赖，DTK 现开始提供 Qt 6 支持。适用于 beta 3 的版本参照如下：

package        | version
---------------|--------
dtkcommon      | 5.6.21
dtkcore        | 5.6.22
dtkgui         | 5.6.22
dtkwidget      | 5.6.22
dtkdeclarative | 5.6.24
qt5integration | 5.6.20
qt5platform-plugins | 5.6.22
dtk6core       | 6.0.4
dtk6gui        | 6.0.5
dtk6widget      | 6.0.4
dtk6declarative | 6.0.7
qt6integration | 6.0.4
qt6platform-plugins | 6.0.4

由于 dtkdeclarative 和 dtkgui 存在一些发布前的紧急修复，所以这两个组件存在版本号未对齐的情况。请直接参照上述表格的版本进行打包，以便确保 dtk 自身互相依赖的版本无误。

关于 qt5platform-plugins，现有的 dwayland 插件可能对非 DDE 环境（例如 KDE）的 wayland 用户存在影响，可参照 [linuxdeepin/developer-center#7217](https://github.com/linuxdeepin/developer-center/issues/7217) 打对应的 patch 规避影响。

dtk6 曾对 Qt 6.2, Qt 6.4 和 Qt 6.6 均进行过适配，但我们目前的研发与测试主要使用 Qt 6.6 版本，故我们建议使用尽可能新的 Qt 6 版本。

目前，使用 dtk6 的正式组件有 dde-application-manager 以及 dde-launchpad，技术预览组件有 dde-shell, treeland。

### dde-application-manager

dde-application-manager 自 1.1.0 版本起进行了完整重构（此次随 beta 3 发布的版本是 1.1.8），接口存在完全不兼容的变动。故原本依赖重构前 dde-application-manager 的应用程序和组件均需要进行更新。比较典型的组件为 dde-dock 与 dde-file-manager。

### dde-launchpad

dde-launchpad 取代了 dde-launcher，原 dde-launcher 由于大量缺陷以及对旧 dde-application-manager 的强依赖，已不再可用。如果您的发行版在使用 dde-launcher，现在可以废弃 dde-launcher 并使用 dde-launchpad 替代了。

尽管在 deepin 中，dde-launchpad 使用的是 Qt 6 构建的版本，dde-launchpad 本身仍然提供了选项来控制使用 Qt 5 或是 Qt 6。我们建议在允许的情况下尽可能使用 Qt 6 而不是 Qt 5，以及无论 Qt 5 或是 Qt 6 支持，请都不要忘记安装 qt5/6integration 插件以及 qt5/6platform-plugins，否则 dde-launchpad 可能会缺失圆角与模糊效果。同样，强烈建议配合最新 DTK 版本构建来获得最佳显示效果。

另外，随 dde-launchpad 一并发布了 dde-application-wizard 项目。此项目是一个守护进程，提供对旧的 dde-application-manager 所提供的卸载服务接口的兼容支持，但可被移植到支持 packagekit 的发行版，故您可以考虑打包移植此组件。如果您的发行版不支持 packagekit，则可考虑暂时 patch dde-launchpad 来隐藏菜单中的卸载按钮，后续 dde-launchpad 会提供 DConfig 来便于发行版定制这些菜单项。

## 技术预览组件

技术预览阶段的组件均可酌情决定是否打包。

### dde-shell

dde-shell 目前是技术预览阶段的项目。dde-shell 旨在将 DDE 桌面环境插件化与模块化，降低开发难度，使各个组件的替换变得更加容易，并且提供更好的桌面环境集成支持。

dde-shell 目前包含 shell 本体，以及预制的新 dock 插件和通知中心插件。

### treeland / ddm

treeland 目前是技术预览阶段的项目，兼顾 Wayland 窗口合成器和显示管理功能。treeland 除需要 dtk6declarative 外，还依赖新项目包括 [waylib](https://github.com/vioken/waylib) 和 [qwlroots](https://github.com/vioken/qwlroots)。需要使用 Qt 版本 >= 6.6.0， wlroots 版本 >= 0.17.0 编译 qwlroots/waylib。

同时我们放弃了对 deepin-kwin wayland 的支持（更新 dde-session 后会屏蔽入口），DDE 所有 wayland 相关的支持均由 Treeland 提供。

### deepin-im

treeland 目前是技术预览阶段的项目，提供了输入法的抽象层。目前，deepin-im 仅期望在 treeland 环境下使用。它会设置 `QT_IM_MODULE` 等环境变量的值来影响实际使用的输入法模块。

## 获取移植帮助

如果您希望得到移植相关的帮助，请考虑加入我们 DDE 移植小组的在线交流群（下列房间有桥接，任选其一即可），一起展开相关的交流：

- Matrix 群：`#dde-port:deepin.org`
- Telegram 群：<https://t.me/ddeport>

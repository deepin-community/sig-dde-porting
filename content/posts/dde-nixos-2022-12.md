---
title: "DDE 移植小组成果展示"
date: 2022-12-12
draft: false
authors: [ "rewine" ]
tags: [ "成果展示" ]
---

自小组成立以来，已经为 DDE 的移植做出了很多贡献，本文做出了一些总结。如有补充或批改，可向 [sig-dde-porting](https://github.com/deepin-community/sig-dde-porting) 提出 pr。

NixOS 的移植从今年 3 月开始，到现在已经有了实用的可能，今后会用一篇文章单独介绍。未来会每月更新进展，关注的同学可以订阅本站 rss。

<!--more-->

### 今年改善可移植性的一些努力

- 修复依赖特定 kwayland 无法在其他发行版编译的问题（为项目增加禁用wayland的option）：
>   deepin-system-monitor，dde-clipboard，dde-session-shell...

- 修复安装路径硬编码问题
>   29 个相关 pr https://github.com/linuxdeepin/developer-center/issues/3167

- 减少代码中的硬编码问题
>    16 个相关 pr https://github.com/linuxdeepin/developer-center/issues/3374  


- 编译 flag 直接覆盖的问题
>    大约 8 处

- 修复 as-need 参数 break 了部分连接器:
>    10 个相关 pr https://github.com/linuxdeepin/developer-center/issues/3345

- 完善项目关于 pkg-config/Config.cmake 的问题(如写死路径，版本号错误，引入依赖检查不完善等等)
>    至少 17 个相关 pr，事实上应该远多于 17，存在多个修改顺便提在一个 pr 的情况。

- 使用 dpkg-architecture 判断架构其他发行版无法使用问题
>    4 处以上

- 避免绝对路径头文件：
>    解决 2 处，但仍有 1 处难以处理（https://github.com/linuxdeepin/dde-session-shell/pull/120）


- dde-kwin 问题
>    [justforlxz](https://github.com/justforlxz) 已经多次修复 dde-kwin 对 kwin 的适配问题，但每次 kwin 每次更新都会重新带来很多问题。正在移植中的 deepin-kwin 将会解决这个麻烦，目前可以使用 Arch（aur）或者 dde-nixos 先行体验。

- 处理 deepin-wallpapers 非自由协议问题（拆包）


### dde-nixos 近期进展

- dtkcore：
   - 其他发行版调用 uosType 直接返回 UosTypeUnknown，避免无意义的报错刷屏
   - 完善 isDDE 函数判断
   - deepin-os-release 判断是其他发行版后不会输出 deepin/uos 特有参数

- dde-control-center:
    - 提供禁用生物认证模块的 option
    - 修复无法设置头像问题
    - 修复密码校验无法通过的问题
    - 修复系统信息/关于本机的显示问题：
        - 启用不应隐藏的“计算机名”，“产品名称”
        - 隐藏“版本”（社区版/专业版）
        - 替换为 NixOS 的 logo
        - 隐藏修改 “计算机名” 的功能
    - 隐藏用户体验模块，禁用对应 dbus 调用，修复通用模块卡钝问题
    - 修复显示内存为 0 的问题

- deepin-movie:
    - 修复格式支持不完整的问题（修复 libPath 硬编码）
    - 修复一个崩溃问题

- dde-grand-search:
    - 修复调用 dde-grand-search-daemon 的权限验证

- deepin-screen-recorder：
    - 修复2个导致崩溃的问题
    - 修复命令行启动无法录屏的问题（dock插件仍然有此问题）

- 修复 GStreamer 音频播放：
    - 需要设置依赖以及 GST_PLUGIN_SYSTEM_PATH_1_0
    - 涉及 deepin-music deepin-voice-note deepin-movie 和 dde-introduction

- deepin-editor：
    - 修复主题路径硬编码造成的界面异常    

- deepin-compressor:
    - 修复无法加载自身插件的问题 

- 修复个别图标丢失的问题（部分 dci 图标需要 qtimageformats 支持）

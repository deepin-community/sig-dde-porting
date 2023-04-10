---
title: "dde-nixos 近期进展公告"
date: 2023-04-10
draft: false
authors: [ "rewine" ]
tags: [ "成果展示" ]
---

目前 dde-nixos 已经分叉，mian 分支进行 v23 的维护，目前主要更新了 dtk 和部分 deepin 开头的应用， dde 开头的核心应用移植暂未实现，dbus 接口不兼容，因此目前不可日常使用。

gomod 分支用于测试使用 buildGoModule 完成构建，仅验证可行性，实际使用还需要调整硬编码相关的 patch。

日常使用 DDE 需要切换 v20 分支，会优先使用已经提交到上游的应用：

```nix
   dde-nixos = {
      url = "github:linuxdeepin/dde-nixos/v20";
      inputs.nixpkgs.follows = "nixpkgs";
    };
```

在 v20 分支，dtk 使用 5.6.3 不再升级，deepin 应用会保持最新的 v20 版的最新版本（不会上 6.0.0），dde 应用冻结为 1 月份打包时测试可用的版本，一般不再升级：

既除了 deepin 应用，其他应用只有在 v23 移植完成后再更新。

目前 NixOS 23.05 — Feature Freeze & Release Blockers 已经开始，进度请关注：
https://github.com/NixOS/nixpkgs/issues/224457#issuecomment-1501383113

向上游贡献的主要调整：

1. 调整 patch

在 dde-nixos 中，编写了 getPatchFrom， replaceAll 等函数帮助 patch 硬编码路径，但打包时为上游添加函数是难以接受的，因此所有的 patch 都需要使用 substituteInPlace 重写：

一个典型的例子是：https://github.com/NixOS/nixpkgs/pull/217806

2. 改善对交叉编译

所有 deepin 启用 strictDeps，调整了 nativeBuildInputs 和 buildInputs 不规范的地方，（使用 qmake 的除外，会造成 qtwebengine 找不到，且上游已经不太关心 qmake）

strictDeps 下无法传播 qtimageformats 问题 ，由 NickCao [解决](https://github.com/NixOS/nixpkgs/pull/213926)

ps：可以使用 `nix-build -A pkgsCross.riscv64.deepin.dtkcore` 尝试交叉编译。目前 x86_64 和 aarch64 可正常编译。


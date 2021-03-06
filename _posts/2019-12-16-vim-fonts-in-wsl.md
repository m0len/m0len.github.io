---
layout: post
title: "解决 WSL 中 Vim 字体异常"
comments: true
date: 2019-12-16
description: ""
categories: [OS]
tags: [wsl, win10, vim]
---

**目录**
* ToC
{:toc}
---

## 问题

在 WSL（Windows Subsystem for Linux）中使用 Vim 时发现字体变成了宋体而不是其他应用中的等线字体，不太协调，所以网上找找资料，找到一个[解决方法](https://0vo.moe/archives/311.html)。但是网站已经打不开了，所以在这里记录一下。

## 步骤

1.用快捷键 `Win + R` 打开“运行”窗口；
2.输入 `regedit` 打开“注册表编辑器”；
3.在上方地址栏输入 `HKEY_CURRENT_USER\Console\` 并回车，
**这里有一个需要注意的地方**，因为原作者使用的是 Ubuntu，所以原文这里是进入 `C:_Program Files_WindowsApps_CanonicalGroupLimited.UbuntuonWindows_1804.2018.817.0_x64__79rhkp1fndgsc_ubuntu.exe` 这里，但因每个人使用的 WSL 发行版不同，比如我使用的 Debian 10 则进入 `C:_Program Files_WindowsApps_TheDebianProject.DebianGNULinux_1.1.7.0_x64__76v4gfsz19hv4_debian.exe` 这里；
4.进入对应路径之后，在右边窗口空白处 `点击右键 > 新建 > DWORD值 > 名称为“CodePage” > 值为十进制的“65001”` ；
5.再次打开发现字体显示已正常。

## 参考

* [Kong's Blog](https://0vo.moe/archives/311.html)
* [Kong's Blog 的 Google 网页快照](https://webcache.googleusercontent.com/search?q=cache:17K7ZCoNiy8J:https://0vo.moe/archives/311.html+&cd=2&hl=zh-CN&ct=clnk)


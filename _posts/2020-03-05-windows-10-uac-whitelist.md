---
layout: post
title: "被 Windows 10 管理员运行搞烦躁？设置 UAC 管理员权限白名单"
comments: true
date: 2020-03-05
description: ""
categories: [OS]
tags: [UAC, windows, win10, windows10, whitelist]
---

**目录**
* ToC
{:toc}
---

## 问题

在使用 Windows 的时候，经常会遇到请求管理员权限的对话框。由于我习惯性使用管理员权限运行 PowerShell ，还有一些需要操作 C 盘的应用也会需要管理员权限，更是经常被它烦扰。不仅如此，使用快捷键来打开需要管理员权限的应用时（我是快捷键爱好者），总是会遇到权限请求对话框无法弹出，需要在任务栏点击呼出的情况，简直雪上加霜。

与（类）Unix 系统不同的是，它只是一个提醒，没有验证用户的作用（后面我发现是可以设置的，下面会说）。所以个人认为，在用户本人自行运行一个程序的情况下，完全没有必要再问一次用户是否批准管理员权限，而应该在用户安装新软件或者在程序运行中需要的权限才应该要求用户注意。经过一番搜索，原来早在 Windows 7 时代就有人提出过解决方案，于是写文记录。

## 完全关闭 UAC

首先，既然它如此“鸡肋”，我们当然可以选择完全关闭它，但是没必要。

关闭它的方法非常简单，使用 Windows 搜索（在开始菜单图标点击右键 --> 搜索），输入“本地安全策略”并回车，进入到本地安全策略窗口。

![](https://github.com/m0len/m0len.github.io/raw/master/img/local-sec-policy.png)

~~打开它的时候你就会遇到一个提权请求。。Emmm~~

依次打开 本地策略 --> 安全选项，在右边窗口下拉，找到“用户账户控制：管理员批准模式中管理员的提升权限提示的行为”并双击，在下拉菜单中选择“不提示，直接提升”。下次就不会弹出确认窗口了。

![](https://github.com/m0len/m0len.github.io/raw/master/img/elevate-wo-prompting.png)

#### 一个发现

在这个下拉菜单中，我发现了一个选项叫做“在安全桌面中提示凭据”，如果选择这项的话，下次提示提升权限的对话框就会要求你输入用户密码或者使用生物认证。对于有指纹识别或者 Windows Hello 的电脑来说，很方便。

## 折中的“白名单”方案

> 既然关掉不好，又不想每次都点击，那么可以通过一个曲线救国的方法来达到“白名单”的效果。

使用 Windows 搜索，输入“任务计划程序”并回车，打开任务计划。

![](https://github.com/m0len/m0len.github.io/raw/master/img/task-sche.png)

先点击一次“任务计划程序库”，然后右键点击它，选择“新文件夹”，起一个喜欢的名字，比如 “UAC Whitelist”。

![](https://github.com/m0len/m0len.github.io/raw/master/img/new-folder.png)

点击刚刚新建的文件夹，在窗口右方点击“创建任务”。**名称**填写一个自己喜欢的，比如 “wl-xxx”。

有一个需要特别注意的地方，这是整件事情的关键。**勾选下方的“使用最高权限运行”**。

配置下拉菜单选择 Windows 10。

![](https://github.com/m0len/m0len.github.io/raw/master/img/new-task.png)

点击上方“操作”标签，点击下方“新建”，选择需要加入“白名单”的软件。

![](https://github.com/m0len/m0len.github.io/raw/master/img/new-task-action.png)

如果是笔记本电脑上的话，需要多一步操作。选择上方“条件”标签，取消勾选下方“只有在计算机使用交流电源时才启动此任务”。

![](https://github.com/m0len/m0len.github.io/raw/master/img/new-task-condition.png)

某些情况下可能需要打开多个窗口，那么可以按如下设置。

![](https://github.com/m0len/m0len.github.io/raw/master/img/alongside-new-instance.png)

## 如何使用

可以在一个专门的地方，比如桌面，右键新建一个“快捷方式”。

![](https://github.com/m0len/m0len.github.io/raw/master/img/wl-new-shortcut.png)

按以下格式填入路径。

```
C:\Windows\System32\schtasks.exe /RUN /TN "文件夹名称\任务名称"
```

比如：

``` 
C:\Windows\System32\schtasks.exe /RUN /TN "UAC Whitelist\wl-xxx"
```

文件夹名称就是刚才新建的文件夹的名称，任务名称就是刚才设置的每个“白名单”的名称，对应填入即可。

双击运行这个快捷方式就会发现，是用管理员权限运行的，而且没有了提权的对话框。


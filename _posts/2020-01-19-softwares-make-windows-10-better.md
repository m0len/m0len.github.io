---
layout: post
title: "一些可以提高 Windows 10 使用效率的软件"
comments: true
date: 2020-01-19
description: ""
categories: [OS]
tags: [windows, software, linux, mac, wsl, everything, sharpkeys, winhotkey, terminal, powershell, aquasnap, powertoys, photon, aria2, vscode, 效率, win10, windows10]
---

**目录**
* ToC
{:toc}
---

> 又名：2019 年我最喜爱的 PC 软件

## 习惯

过去的五年，我的主力设备是一台 4GB 内存的 MacBook Air。到现在，日常工作都已经不堪用了，毅然换成了一台 16GB 内存的 Windows 10 笔记本，于是开始了重走 Windows 之路。在用 Mac 期间，我也接触过不少 Linux 的发行版，还玩过不少服务器。现在主力台式机都还是用 Debian 10，给予了我的业余爱好极大便利。回想这几年的知识积累，主要都来源于 Mac 和 Linux。

阔别五年，Windows 系统的改变是惊人的。对专业使用者更友好的 WSL、Powershell Core 和 Hyper-V，更好用的全局搜索以及更合理的触摸板手势等，使得 Win 的体验与 Mac 的差距拉近了不少。换了平台之后，许多习惯想要保留，想看能否找到 Win 下的软件替代品。于是从拿到新电脑的第一天起，我就打算依照我使用 Mac 和 Linux 的习惯来武装 Windows 10，试图打造用起来更顺手的 Windows 10。 

这是我重新学习 Windows 系统的一篇记录。

## 需求 —— 替代

### Time Machine —— 文件历史记录

Mac 机上的杀手级应用之一，开启之后会每隔一段时间备份用户电脑磁盘上的所有文件，第一次使用的体验就真的像在坐时光机器那般美妙，因为实在是太无感和舒心的体验了。一旦遇上病毒删除或者加密了文件，又或是遇上磁盘损坏等情况，都可以恢复备份过的任一时间段里的文件。即使是没有这么极端的情况，比如修改了文件没有保存，都可以通过“乘坐”时光机器“回到过去”，那种感觉只有丢过数据的人才能明白。

于是在换电脑之后的第一天，我的心就像没有底一样，惶惶不可终日。在想什么时候磁盘会突然坏掉，或者什么时候就中了勒索病毒把我的文件都加密了，于是我开始寻找时光机器在 Windows 上的替代品。经过一番寻觅，我在控制面板上找到一丝蛛丝马迹，即系统自带的**文件历史记录**。再经过一番摸索和实验，我终于放心地把用户文件夹里的所有文件的备份任务托付给它。

**文件历史记录**属于增量备份，而且会检测每一个文件在上次备份与本次备份之间有没有修改过，如果没有修改过的话是不会重复备份的，节省空间。开启的方法也十分简单，而且恢复文件的操作同样简洁明了，个人感觉甚至比 Time Machine 上那科幻而又不明所以的恢复界面直观很多。

首先打开控制面板：
![](https://github.com/m0len/m0len.github.io/raw/master/img/control-panel.png)

点击“通过文件历史记录……”进入文件历史记录的设置：
![](https://github.com/m0len/m0len.github.io/raw/master/img/file-history-record-interface.png)

Windows 规定需要把文件历史记录保留在非本地磁盘的存储设备上，这点比 Time Machine 要好一点，更倾向于文件备份而不是单纯的版本控制（Time Machine 允许把文件记录保存在本地磁盘上，磁盘损坏会导致所谓“备份”的文件一并消失）。如果电脑接上了移动存储设备的话这里会自动列出，通过点击左侧的**选择驱动器**也可以选择网络存储设备。这里只要点击**开启**，就会设置默认每小时备份一次，默认永久保留所有文件历史记录。
![](https://github.com/m0len/m0len.github.io/raw/master/img/file-history-record-on.png)

![](https://github.com/m0len/m0len.github.io/raw/master/img/file-history-record-select-driver.png)

通过高级设置可以选择保存的版本数以及备份的频率等：
![](https://github.com/m0len/m0len.github.io/blob/master/img/file-history-record-advanced-settings.png)

万一遇到不测，需要恢复文件的话有两种途径。一是进入文件历史记录的设置页，选择**还原个人文件**，这种方式比较像 Time Machine 上的选择时间窗口的形式，比较方便查看备份过的所有文件版本：
![](https://github.com/m0len/m0len.github.io/raw/master/img/file-history-record-recover.png)

而第二种方式适合直接恢复单个文件，即在文件夹中直接右键想要恢复的文件，选择**还原以前的版本**：
![](https://github.com/m0len/m0len.github.io/raw/master/img/file-recover.png)

然后在对话框中选择想要恢复到的文件版本，选择时还能通过打开文件查看文件内容来作决定：
![](https://github.com/m0len/m0len.github.io/raw/master/img/file-recover-select-version.png)

### Spotlight —— Everything

Mac 机上的杀手级应用之二，可以搜索电脑上的一切应用、文件，速度极快，比 Win 下的文件管理器里的搜索好用太多。但是别担心，Win 上早就有类似的应用了，那就是大名鼎鼎的 **Everything**（[官网](https://www.voidtools.com/)）。

![](https://github.com/m0len/m0len.github.io/raw/master/img/everything-interface.png)

主界面简洁明了，所以打开速度也很快，能媲美系统原生应用。搜索最重要的就是索引，**Everything** 建立索引的速度也非常快，即使是新的文件也可以马上被搜索到，无需像 Windows 资源管理器的搜索一样等待进度条。

### Command-Alt —— SharpKeys

![](https://github.com/m0len/m0len.github.io/raw/master/img/sharpkeys-interface.png)

这应该是一个极少数人需要的功能，姑且算是一个伪需求。**SharpKeys**（[官网](https://github.com/randyrants/sharpkeys)）是一个简单的小软件，开源。功能也简单，就是按键映射。我实际上只用到它一个功能————修改 `Ctrl` 键和 `Alt` 键的位置，就是把左 `Ctrl` 键映射成左 `Alt` 键，把左 `Alt` 键映射成左 `Ctrl` 键。这听起来十分无谓，但其实并不是。这个习惯来源于 Mac 机。用过 Mac 机的人都知道，Mac 机有 `Command` 键，功能相当于 Win 的 `Ctrl` 键，用作组合快捷键，而 Mac 机上的 `Command` 键就在左手拇指的地方，而 Win 的 `Crtl` 键在左手小拇指的地方，在双手正常打字姿势下十分尴尬，难以使用，导致 Win 的快捷键使用效率非常低，这就是这个软件的存在意义之一。如果还要问为什么的话，借用某 Blogger 的一句话：

> [Because Mac OS is better. It just is.————Rio Weber](https://medium.com/riow/make-alt-key-ctrl-on-windows-10-just-like-mac-c1726703e412)

### Terminal —— WSL

![](https://github.com/m0len/m0len.github.io/raw/master/img/wsl-interface.png)

在我看来，这是 Windows 10 的杀手级应用。

忘记了从哪一年开始，蓝色巨人微软开始拥抱开源，第一件大事是收购 GitHub。而在 Windows 里塞入一个 Linux 系统就是第二件大事。占据全世界绝大部分消费市场的闭源操作系统整合了一个全世界使用最广泛的开源操作系统，这在当年是完全无法想象的。这就像是有史以来第一次，Developers 和 Gamers 达成了共识。

虽然目前的 WSL（[官网](https://docs.microsoft.com/en-us/windows/wsl/install-Win10)）还存在不少的问题，比如没有完全的操作系统权限（比如无法调用某些硬件）、不能自编译内核（只能用内置的微软编译的内核）等等，它并不是一个类似 KVM 般自由的虚拟机。但是据说这些问题会在不久之后的 WSL 2 中得到解决。本人表示非常期待这基于 Hyper-V 的 WSL 2。

说回 WSL，它就像是一个 Windows 10 外挂，既独立又共存。独立是因为它有自己的文件系统，可以与宿主的文件系统分别管理；共存是因为它带来了一大堆 Linux 命令，用于管理和操作宿主机的文件，这给了 Windows 素来不好用的终端带来了质的改变。在熟悉 Mac 和 Linux 的使用者面前，就像使用 Terminal 一样方便，这感觉更像是给 Android 安装了 BusyBox 一样。同时，Win 和 WSL 的文件还可以互访，win 下进入 `$wsl` 就可以管理 WSL 的文件目录，而宿主机的硬盘则是挂载在 WSL 里的 `/mnt` 。

不说别的，在不想费时间打开虚拟机的场合，有一个可以立刻使用的 Linux 对于使用者来说也非常方便。

### Split View —— AquaSnap、PowerToys

![](https://github.com/m0len/m0len.github.io/raw/master/img/aquasnap-interface.png)

分屏，应该是每个现代操作系统必备了。诸如屏幕越来越大，分屏提高工作效率之类的陈词滥调都无需再提，~~写代码时开着 Google/StackOverflow 刚需~~。

**AquaSnap**（[官网](https://www.nurgo-software.com/products/aquasnap)）带来的功能还远不止简单的分屏，但我用得最多的功能反而是它提供的众多窗口操作快捷键。比如我最常用的外接屏幕的窗口移动，AquaSnap 提供了一个 `Shift` + `Win` + 方向键的按键组合，再也不用搓触控板了；还有窗口居中的快捷键 `Ctrl` + `Win` + `C` ，拯救了强迫症； `Win` + `Enter` 快捷键，可以让当前窗口最大化，让屏幕全力发挥；还能通过 `Win` + `Esc` 快速关闭当前窗口等。

说到分屏，还有一个不得不提的微软自产小软件————**PowerToys**（[官网](https://github.com/microsoft/PowerToys)），它提供了一大堆好用又轻巧的功能。这里说的是分屏功能————FancyZones。

![](https://github.com/m0len/m0len.github.io/raw/master/img/fancyzones-interface.png)

在这里，可以设置各种不同的分屏布局，从数量到样式，还可以套用自带的模板，快速设置分屏形式。

### Keyboard Shortcuts —— WinHotkey

在 Linux 某些发行版的桌面坏境中（比如 GNOME），可以设置一些键盘快捷键，如最常用的 `Ctrl` + `Alt` + `T` 打开终端，是非常实用且使用率很高的设置。在 Win 下，也有一个类似的应用，**WinHotkey**（[官网](https://directedge.us/content/winhotkey)）。

![](https://github.com/m0len/m0len.github.io/raw/master/img/winhotkey-interface.png)

这个软件的功能和它的名字一样简单直白，没有任何花哨的界面和不相干的功能，它能且只能配置快捷键，让你快速地打开一些应用、文件或网址。

### Aria2 —— Photon

![](https://github.com/m0len/m0len.github.io/raw/master/img/photon-interface.png)

Aria2 是一个开源的下载工具，支持种子、HTTP 等协议。还能通过伪装 BT 客户端，用于 PT。但它只有命令行界面，使用起来不是特别方便，于是很多人为其开发了图形界面，**Photon**（[官网](https://github.com/alanzhangzm/Photon)）就是其中之一。在 Linux 下使用的话可以用 Docker 来搭建，非常方便。说到这里，必须要吐槽一个 Windows 10 的问题。在 Windows 10 下，Docker 的实现方法是 Hyper-V 的一个虚拟机（Linux 虚拟机，可以设置为 Windows 虚拟机，但是很多 Docker 镜像都是基于 Linux 的，这还怎么用？），而 Hyper-V 和 其他虚拟机如 Virtual Box 又有冲突，权衡过后我放弃了 Hyper-V，也就放弃了 Docker for Windows，所以只能找一个图形客户端来用 Aria2 了。

### CLI —— PowerShell Core 6

![](https://github.com/m0len/m0len.github.io/raw/master/img/powershellcore-interface.png)

个人认为，不算上 WSL 的话，这就是 Win 下最好的命令行工具。不过要注意的是，这不是 Windows 10 自带的那一个，这个需要单独从[官网](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-windows)下载，功能与自带的也有所不同。

这里相关的美化可以参考[这里](https://blog.walterlv.com/post/beautify-powershell-like-zsh.html)。

这是我用过最顺手的 Windows 命令行，相比之下传统的 CMD 简直是半成品。但与 Mac 和 Linux 相比的话还是有很大差距，希望拥抱开源的微软可以持续优化吧。

### Editor —— Visual Studio Code

![](https://github.com/m0len/m0len.github.io/raw/master/img/vscode-interface.png)

一个万能编辑器，也是我正用来写博客的工具。借助数量庞大的插件，它可以支持目前任何一种主流或非主流的编程语言、脚本语言，甚至是 Markdown（写 Markdown 舒适）。它就是微软自家的 Visual Studio Code（[官网](https://code.visualstudio.com/)）。这个应该不需要多介绍了，Google 一下几乎全是赞美之言，而且它正变得越来越出名，关键它还是开源的。

### Finder —— Winaero Tweaker

![](https://github.com/m0len/m0len.github.io/raw/master/img/winaero-interface.png)

Window 10 默认的文件管理器界面是“库”，相当于一个文件夹的集合界面，是一个挺好用的快速打开文件夹的功能。但是很可惜的是它并没有给我们选择哪些文件夹应该出现在“库”的自由，要知道 Mac 和 Linux 都是可以自己添加文件夹到左侧边栏上作为快捷方式的。于是在网上搜索一番找到这么一个免费软件，功能十分多，可以自定义电脑的许多东西，但是这里用到的是其中一个**修改此电脑**的功能，如下图：

![](https://github.com/m0len/m0len.github.io/raw/master/img/winaero-this-pc.png)

通过这个功能，可以将任何文件夹添加到 Window 文件管理器的“库”中，还能自定义图标和显示名称。将常用的文件夹添加进去之后，使用电脑就更顺手了。

### 结语

透过以上软件，我基本原样复制了 Mac 和 Linux 上所有的优秀体验。但是要和 Mac 相比，还是存在挺大差距。但是不可否认的是，Windows 确实在变好，只是还算不上优秀。希望这些软件能让 Windows 10 更好用吧。


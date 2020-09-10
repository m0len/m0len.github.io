---
layout: post
title: "手动设置 Debian 10 硬盘分区并使用 LUKS 加密"
comments: true
date: 2019-12-26
description: ""
categories: [OS]
tags: [debian, lvm, luks, encrypt]
---

---
* ToC
{:toc}
---

> 本方案是不加密 ESP 和 BOOT 分区的方案，相对比较简单。需要加密 BOOT 分区的话可以参考另外的教程。

## 分区方案

![00plan](https://github.com/m0len/m0len.github.io/raw/master/img/lukslvm.png)

## 选择手动分区

首先选择**图形安装（Graphic Install）**。

![01start](https://github.com/m0len/m0len.github.io/raw/master/img/01start.png)

之前的步骤先省略，到这一步的时候选择**手动（Manual）**。

![02manual](https://github.com/m0len/m0len.github.io/raw/master/img/02manual.png)

选择要把 Debian 安装在哪个**物理硬盘**上。

![03choose-disk](https://github.com/m0len/m0len.github.io/raw/master/img/03choose-disk.png)

确认**创建新的空分区表**。

![04create-new-empty-partition-table](https://github.com/m0len/m0len.github.io/raw/master/img/04create-new-empty-partition-table.png)


## 创建 `EFI 分区`（`ESP`）

选择**自由空间（Free Space）**。

![05select-free-space](https://github.com/m0len/m0len.github.io/raw/master/img/05select-free-space.png)

由于使用 UEFI 引导，所以先**创建 ESP 空间**。这里选择创建新分区。

![06create-new-partition](https://github.com/m0len/m0len.github.io/raw/master/img/06create-new-partition.png)

空间按需填写，我这里分了 500 MB。

![07edit-size-esp](https://github.com/m0len/m0len.github.io/raw/master/img/07edit-size-esp.png)

选择用于 **ESP（EFI System Partition）** 分区。然后点选完成设定分区。

![08use-as-esp](https://github.com/m0len/m0len.github.io/raw/master/img/08use-as-esp.png)

回到刚才的菜单发现已经多了一个 **ESP 分区**。

![09choose-free-space](https://github.com/m0len/m0len.github.io/raw/master/img/09choose-free-space.png)


## 创建 `/boot` 分区

同理，点选**自由空间（Free Space）**。选择用作 **ext2** 文件系统，并**挂载在 `/boot`**。用作引导启动。

![10use-as-ext2-and-mount-at-boot](https://github.com/m0len/m0len.github.io/raw/master/img/10use-as-ext2-and-mount-at-boot.png)

回到硬盘分区界面可以看到已经多了一个 `/boot` 分区。

![11configure-enc-vols](https://github.com/m0len/m0len.github.io/raw/master/img/11configure-enc-vols.png)


## 创建使用 LUKS 加密的 LVM 并分配空间

到这里，就要开始**创建加密卷**了，选择上面的**配置加密卷（Configure encrypted volumes）**。

选择**创建加密卷**。

![12create-enc-vols](https://github.com/m0len/m0len.github.io/raw/master/img/12create-enc-vols.png)

选择**所有剩余的空间**。

![13choose-the-rest-free-space](https://github.com/m0len/m0len.github.io/raw/master/img/13choose-the-rest-free-space.png)

详细**配置加密选项**，一般保持默认即可。

![14done-setting](https://github.com/m0len/m0len.github.io/raw/master/img/14done-setting.png)

确认**写入磁盘并配置加密卷**。

![15confirm-write-to-disk](https://github.com/m0len/m0len.github.io/raw/master/img/15confirm-write-to-disk.png)

选择**完成**。

![16finish-setting-up-enc-vols](https://github.com/m0len/m0len.github.io/raw/master/img/16finish-setting-up-enc-vols.png)

这里选择 Yes 的话就会开始往加密卷写入随机数据以确保磁盘原数据无法被恢复。**这一步之后就无法回头了。**

![17erase-data-on-enc-vol](https://github.com/m0len/m0len.github.io/raw/master/img/17erase-data-on-enc-vol.png)

等待写入随机数据。这一步根据设备不同可能需要特别长的时间。

![18wait-for-erase-or-just-click-cancel](https://github.com/m0len/m0len.github.io/raw/master/img/18wait-for-erase-or-just-click-cancel.png)

设定刚刚配置的**加密卷的密码**，开机挂载的时候需要用到。

![19create-passphrase](https://github.com/m0len/m0len.github.io/raw/master/img/19create-passphrase.png)


![20configure-logic-vol-manager](https://github.com/m0len/m0len.github.io/raw/master/img/20configure-logic-vol-manager.png)

至此，加密卷创建完成。接下来就是创建 Debian 系统的其他分区，包括根分区，根目录，家目录，交换空间等等。这里选择**配置逻辑卷管理器**。

### 创建逻辑卷并挂载于 `/` 作为根目录

这里选择**创建卷组**，一个卷组下可以创建多个逻辑卷。

![21create-lvm-group](https://github.com/m0len/m0len.github.io/raw/master/img/21create-lvm-group.png)

这里选择在刚刚创建的**加密卷中创建卷组**。并且可以看到加密卷是 ext4 格式的。

![22click-on-enc-vol-just-created](https://github.com/m0len/m0len.github.io/raw/master/img/22click-on-enc-vol-just-created.png)

创建好卷组之后就**创建逻辑卷**。

![23create-logical-vol](https://github.com/m0len/m0len.github.io/raw/master/img/23create-logical-vol.png)

逻辑卷自然是创建在**刚刚创建的卷组内**。

![24select-lvm-group-just-created](https://github.com/m0len/m0len.github.io/raw/master/img/24select-lvm-group-just-created.png)

这里按需**选择所需空间**。由于我选择先创建用于交换空间的逻辑卷，所以按照物理内存输入所需。

![25use-as-swap-and-choose-size](https://github.com/m0len/m0len.github.io/raw/master/img/25use-as-swap-and-choose-size.png)

**创建更多的逻辑卷**。这里创建用于根分区的空间，所以选择余下所有空间。

![26more-logical-vols](https://github.com/m0len/m0len.github.io/raw/master/img/26more-logical-vols.png)

这里每个人不一样，如果需要家目录和 `/var` 等分开的话可以继续创建逻辑卷，否则选择**完成**。

![27finish-creating-lvm](https://github.com/m0len/m0len.github.io/raw/master/img/27finish-creating-lvm.png)

这里可以看到上面多了刚刚创建的两个逻辑卷。

![28lvm-created](https://github.com/m0len/m0len.github.io/raw/master/img/28lvm-created.png)

此时，按照刚刚的规划**点选并配置逻辑卷**。

![29select-partition-for-root-dir](https://github.com/m0len/m0len.github.io/raw/master/img/29select-partition-for-root-dir.png)

这一个我选择用于 **ext4** 文件系统并挂载于 `/`。

![30use-as-ext4-and-mount-at-root](https://github.com/m0len/m0len.github.io/raw/master/img/30use-as-ext4-and-mount-at-root.png)


### 创建逻辑卷并用作 `swap`

同理，**选择并配置另一个逻辑卷**。

![31select-partition-for-swap](https://github.com/m0len/m0len.github.io/raw/master/img/31select-partition-for-swap.png)

我这里选择用于**交换空间**。

![32use-as-swap](https://github.com/m0len/m0len.github.io/raw/master/img/32use-as-swap.png)

至此，分区完成了。点选**完成分区并写入磁盘**。

![33finish-partition](https://github.com/m0len/m0len.github.io/raw/master/img/33finish-partition.png)

最后**确认一下分区方案并确定**。

![34confirm-writing-to-disk](https://github.com/m0len/m0len.github.io/raw/master/img/34confirm-writing-to-disk.png)


![35wait-for-installation](https://github.com/m0len/m0len.github.io/raw/master/img/35wait-for-installation.png)

接下来就是剩余的安装系统步骤。至此，手动设置 Debian LUKS 加密 LVM 完成。

---
layout: post
title: "安装 Debian 10 到 ZFS Mirror 阵列"
comments: true
date: 2020-10-05
description: ""
categories: [OS]
tags: [debian, zfs, zpool, installation, encryption]
---

**目录**
* ToC
{:toc}
---

> 本文参考了 [OpenZFS 官方提供的教程](https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Buster%20Root%20on%20ZFS.html) ，并加入自己的想法。以下步骤非常繁琐和复杂，任何遗漏和错误的输入都可能导致无法挽回的后果。任何不明白的操作请以官方教程和 [Troubleshooting](https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Buster%20Root%20on%20ZFS.html#troubleshooting) 为主。

## Why

最近 [ServerTheHome(STH)](https://www.servethehome.com/introducing-project-tinyminimicro-home-lab-revolution/) 介绍了一个新企划，称为 TinyMiniMicro Project。主要就是使用一些便宜的迷你桌面主机（Desktop Mini），组成集群来达成比相对昂贵的服务器更高的性能。

我看到这个计划的时候可以说相当惊喜，作为小钢炮爱好者，一直想组一台 HP 或 DELL 的 Desktop Mini 作为具有便携性足够高的备份装置和私人服务器（NAS + Server）。这样既可以定时备份电脑和手机的数据，亦可以在空闲时备份到云端 AWS Glacier 或者 GCP Coldline，加上可以运行一些自制小玩意，搬家时又可以少一点痛苦。

但是迷你主机其中一个显而易见的缺点就是通常只有一个 2.5 英寸的硬盘位，中高端型号可能会多一个 m.2 硬盘位。相比硬盘位的不足，迷你主机一般会提供很多 USB 接口，中高端型号前后加起来基本会有 6 个 USB 3.0 接口，可以接移动硬盘来组磁盘阵列，也可以接 U 盘来装系统，使用内置磁盘用于磁盘阵列。

## How

我打算使用的方案是使用至少两个 U 盘，使用 ZFS 文件系统组成磁盘镜像（Mirror，相当于RAID-1）安装系统，然后内部使用 2.5 英寸磁盘加上 m.2 SSD 同样组成 ZFS Mirror存放备份的数据，对于我的使用已经能够提供足够的冗余了。

### 一点说明

需要说明的是，这里所说的数据盘有点区别。我打算对磁盘数据进行加密，开机就需要输入密码解锁磁盘加密的话需要外接显示器和键盘。但由于这是一个无头服务器，显然这不可能。因此，该所谓的数据盘是作为另一个真正用户（称为 `realuser` ）的 `home` 目录使用，而默认的用户（称为 `ongo` ）仅用于 SSH 登录或挂载数据盘。

具体的性能表现我没有测试过，这里只说方法。USB 3.0 的带宽理论上是足够系统运行的，当然使用的 U 盘不能太慢。相比于性能，U 盘更需要担心的是读写寿命，这是我需要使用 Root on ZFS 方案的原因。

### 示意图

![](https://github.com/m0len/m0len.github.io/raw/main/img/rootonzfsdebian.png)

本文将分成上、下部分。上半部分介绍如何将 Debian 10 Buster 安装在 ZFS Mirror 上，下半部分介绍如何在无头服务器上方便地使用加密家目录。

## 准备工作

**1. 下载 Debian 10 Live 镜像**

[https://www.debian.org/CD/live/](https://www.debian.org/CD/live/)

**2. 进入 Live 环境，并配置好网卡和网络连接**

**3. 以下所有步骤，包括磁盘分区、系统安装都在 Live 系统下进行**

**4. 在 Live 环境安装需要用到的包**

``` 
apt install --yes debootstrap gdisk dkms dpkg-dev linux-headers-$(uname -r)
```

``` 
apt install --yes -t buster-backports --no-install-recommends zfs-dkms
```

启用 ZFS 模块

``` 
modprobe zfs
```

``` 
apt install --yes -t buster-backports zfsutils-linux
```

*以上步骤会安装 DKMS 并使用其管理 ZFS 模块。

## 磁盘分区

本文共使用 U 盘 2 个（ `/dev/sda` , `/dev/sdb` )，作为系统盘（以下称为 `/dev/sdX` ）。2.5 SATA 磁盘 1 个（ `/dev/sdc` ），m.2 SSD 1 个（ `/dev/sdd` ），作为数据盘（以下称为 `/dev/sdY` ）。

**进行任何磁盘操作前请确认选择了正确的磁盘，否则将带来严重后果。**

**1. 确认无误后，删除所有磁盘的所有分区，请提前备份好数据。**

*该操作需要对**所有磁盘**执行。

``` 
sgdisk --zap-all /dev/sdX
sgdisk --zap-all /dev/sdY
...
```

**2. 磁盘分区**

**创建大小为 512 MB 的 EFI 分区**

*该操作需要对**所有系统盘**执行。

``` 
sgdisk     -n2:1M:+512M   -t2:EF00  /dev/sdX
...
```

**创建大小为 1 GB 的 `/boot` 分区**

*该操作需要对**所有系统盘**执行。

``` 
sgdisk     -n3:0:+1G      -t3:BF01  /dev/sdX
...
```

**创建大小为所有剩余空间的 `/` 分区**

*该操作需要对**所有系统盘**执行。

``` 
sgdisk     -n4:0:0        -t4:BF00  /dev/sdX
...
```

## 创建作为 `/boot` 分区的池（boot pool），称为 `bpool`

*该操作需要对**加入 Mirror 的所有系统盘的第 3 个分区**执行，如 `/dev/sda3` 和 `/dev/sdb3` 。

``` 
zpool create \
    -o ashift=12 -d \
    -o feature@async_destroy=enabled \
    -o feature@bookmarks=enabled \
    -o feature@embedded_data=enabled \
    -o feature@empty_bpobj=enabled \
    -o feature@enabled_txg=enabled \
    -o feature@extensible_dataset=enabled \
    -o feature@filesystem_limits=enabled \
    -o feature@hole_birth=enabled \
    -o feature@large_blocks=enabled \
    -o feature@lz4_compress=enabled \
    -o feature@spacemap_histogram=enabled \
    -o feature@zpool_checkpoint=enabled \
    -O acltype=posixacl -O canmount=off -O compression=lz4 \
    -O devices=off -O normalization=formD -O relatime=on -O xattr=sa \
    -O mountpoint=/boot -R /mnt \
    bpool mirror \
    /dev/sdX3 \
    /dev/sdX3
```

## 创建作为 `/` 分区的池（root pool），称为 `rpool`

*该操作需要对**加入 Mirror 的所有系统盘的第 4 个分区**执行，如 `/dev/sda4` 和 `/dev/sdb4` 。

*由于使用无头服务器，本文没有使用 ZFS 提供的加密，有需要的话可以去官方教程处研究。

``` 
zpool create \
    -o ashift=12 \
    -O acltype=posixacl -O canmount=off -O compression=lz4 \
    -O dnodesize=auto -O normalization=formD -O relatime=on \
    -O xattr=sa -O mountpoint=/ -R /mnt \
    rpool mirror \
    /dev/sdX4 \
    /dev/sdX4
```

## 安装最小（minimal）系统

**1. 创建 ZFS filesystem dataset 作为容器**

``` 
zfs create -o canmount=off -o mountpoint=none rpool/ROOT
zfs create -o canmount=off -o mountpoint=none bpool/BOOT
```

**2. 为 `/root` 和 `/boot` 创建 filesystem dataset**

``` 
zfs create -o canmount=noauto -o mountpoint=/ rpool/ROOT/debian
zfs mount rpool/ROOT/debian

zfs create -o mountpoint=/boot bpool/BOOT/debian
```

**3. 创建 dataset**

*必选

``` 
zfs create                                 rpool/home
zfs create -o mountpoint=/root             rpool/home/root
zfs create -o canmount=off                 rpool/var
zfs create -o canmount=off                 rpool/var/lib
zfs create                                 rpool/var/log
zfs create                                 rpool/var/spool
```

*可选

``` 
# 将 `/var/cache` 和 `/var/tmp` 排除在自动快照外
zfs create -o com.sun:auto-snapshot=false  rpool/var/cache
zfs create -o com.sun:auto-snapshot=false  rpool/var/tmp
chmod 1777 /mnt/var/tmp
```

``` 
zfs create                                 rpool/opt
zfs create                                 rpool/srv
zfs create -o canmount=off                 rpool/usr
zfs create                                 rpool/usr/local
zfs create                                 rpool/var/www

# 如需安装 Docker 并需要独立 dataset 和快照
zfs create -o com.sun:auto-snapshot=false  rpool/var/lib/docker

# 创建 `/tmp` 并将 `/tmp` 排除在自动快照外，回滚 filesystem 时就不会回滚用户数据
zfs create -o com.sun:auto-snapshot=false  rpool/tmp
chmod 1777 /mnt/tmp
```

**4. 安装最小系统（minimal system）**

``` 
debootstrap buster /mnt
```

## 配置系统

**1. 修改 hostname**

``` 
echo HOSTNAME > /mnt/etc/hostname
vi /mnt/etc/hosts
```

**2. 配置网卡**

``` 
# 找出网卡名称
ip addr show

# 写入配置
vi /mnt/etc/network/interfaces.d/NAME

# 基本网卡配置
auto NAME
iface NAME inet dhcp
```

**3. 配置安装包源**

``` 
vi /mnt/etc/apt/sources.list

deb http://deb.debian.org/debian buster main contrib
deb-src http://deb.debian.org/debian buster main contrib
```

``` 
vi /mnt/etc/apt/sources.list.d/buster-backports.list

deb http://deb.debian.org/debian buster-backports main contrib
deb-src http://deb.debian.org/debian buster-backports main contrib
```

``` 
vi /mnt/etc/apt/preferences.d/90_zfs

Package: libnvpair1linux libuutil1linux libzfs2linux libzfslinux-dev libzpool2linux python3-pyzfs pyzfs-doc spl spl-dkms zfs-dkms zfs-dracut zfs-initramfs zfs-test zfsutils-linux zfsutils-linux-dev zfs-zed
Pin: release n=buster-backports
Pin-Priority: 990
```

**4. 绑定虚拟Live 系统的文件系统并改变根目录（ `chroot` ）到新系统**

``` 
mount --rbind /dev  /mnt/dev
mount --rbind /proc /mnt/proc
mount --rbind /sys  /mnt/sys
chroot /mnt /usr/bin/env DISK=/dev/sdX bash --login
```

**5. 配置基本系统环境**

``` 
ln -s /proc/self/mounts /etc/mtab
apt update
```

配置语言和时区

``` 
apt install --yes locales
dpkg-reconfigure locales
dpkg-reconfigure tzdata
```

**6. 在新系统中安装 ZFS**

``` 
apt install --yes dpkg-dev linux-headers-amd64 linux-image-amd64
apt install --yes zfs-initramfs
echo REMAKE_INITRD=yes > /etc/dkms/zfs.conf
```

**7. 安装 GRUB**

*该操作只需要对**加入 Mirror 的 1 个系统盘的第 2 个分区**执行，如 `/dev/sda2` 。

``` 
apt install dosfstools
mkdosfs -F 32 -s 1 -n EFI /dev/sdX2
mkdir /boot/efi
echo PARTUUID=$(blkid -s PARTUUID -o value /dev/sdX2 \
   /boot/efi vfat nofail,x-systemd.device-timeout=1 0 1 >> /etc/fstab
mount /boot/efi
apt install --yes grub-efi-amd64 shim-signed
```

**8. 删除不需要的 `os-prober` **

``` 
dpkg --purge os-prober
```

**9. 设置 root 密码**

``` 
passwd
```

**10. 设置启动时 import bpool**

``` 
vi /etc/systemd/system/zfs-import-bpool.service

[Unit]
DefaultDependencies=no
Before=zfs-import-scan.service
Before=zfs-import-cache.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/sbin/zpool import -N -o cachefile=none bpool

[Install]
WantedBy=zfs-import.target
```

设置启动时 import

``` 
systemctl enable zfs-import-bpool.service
```

## 安装 GRUB

**1. 确认 ZFS boot filesystem 有被正确识别到**

``` 
grub-probe /boot
```

**2. 刷新 initrd**

``` 
update-initramfs -c -k all
```

**3. 如果想关掉 ZFS 的 ‘missing feature’ 提醒**

``` 
vi /etc/default/grub

# 修改如下一行 
GRUB_CMDLINE_LINUX="root=ZFS=rpool/ROOT/debian"
```

**4. 更新 GRUB**

``` 
update-grub
```

**5. 安装 bootloader**

``` 
grub-install --target=x86_64-efi --efi-directory=/boot/efi \
    --bootloader-id=debian --recheck --no-floppy
```

**6. 修复 ZFS filesystem 挂载顺序**（很重要的一步，如果之前的步骤有漏掉这步会完成不了）

``` 
mkdir /etc/zfs/zfs-list.cache
touch /etc/zfs/zfs-list.cache/bpool
touch /etc/zfs/zfs-list.cache/rpool
ln -s /usr/lib/zfs-linux/zed.d/history_event-zfs-list-cacher.sh /etc/zfs/zed.d
zed -F &
```

**确认 `zed` 更新了 ZFS 缓存。如果执行以下两行输出结果，则成功；如无结果输出，则失败，检查之前步骤哪里有漏。**

``` 
cat /etc/zfs/zfs-list.cache/bpool
cat /etc/zfs/zfs-list.cache/rpool
```

**如果以上两行命令任何一行输出结果为空，则强制更新缓存并重新执行以上两行命令。**

``` 
zfs set canmount=on     bpool/BOOT/debian
zfs set canmount=noauto rpool/ROOT/debian
```

**如果成功，则停止 `zed`**

``` 
fg
按 `Ctrl-C`
```

**修复路径，清除 `/mnt`**

``` 
sed -Ei "s|/mnt/?|/|" /etc/zfs/zfs-list.cache/*
```

## 安装完成，首次启动

**1. 保存安装的 snapshot**

``` 
zfs snapshot bpool/BOOT/debian@install
zfs snapshot rpool/ROOT/debian@install
```

以后每次更新前最好都保存一次快照并删除旧快照。

**2. 退出 `chroot` 环境， 回到 Live 环境**

``` 
exit
```

**3. 卸载所有文件系统和 ZFS**

``` 
mount | grep -v zfs | tac | awk '/\/mnt/ {print $3}' | \
    xargs -i{} umount -lf {}
zpool export -a
```

**4. 重启**

``` 
reboot
```

**5. 创建一个普通用户（本例中为： `ongo` ，具体自定）**

``` 
# 将 `username` 改为自己需要的
zfs create rpool/home/username
adduser username

cp -a /etc/skel/. /home/username
chown -R username:username /home/username
usermod -a -G audio,cdrom,dip,floppy,netdev,plugdev,sudo,video username
```

**6. 在所有 Mirror 磁盘中安装 GRUB**

``` 
# 卸载 efi 分区
umount /boot/efi
```

``` 
# if 选择前面安装 GRUB 时选择的磁盘，of 选择 Mirror 中的其他磁盘，该 `dd` 命令总共需要执行（n-1）次，n 为 Mirror 中磁盘数量。
dd if=/dev/sdX2 \
   of=/dev/sdX2

# 该命令的磁盘对应上一条 `dd` 命令中的 of 对应的磁盘，注意是整盘，如 `/dev/sdb` ，不是分区。
efibootmgr -c -g -d /dev/sdX \
    -p 2 -L "debian-2" -l '\EFI\debian\grubx64.efi'

# 挂载 efi 分区
mount /boot/efi
```

**7. 可选：配置独立的 SWAP 分区**

``` 
zfs create -V 4G -b $(getconf PAGESIZE) -o compression=zle \
    -o logbias=throughput -o sync=always \
    -o primarycache=metadata -o secondarycache=none \
    -o com.sun:auto-snapshot=false rpool/swap
```

**配置 SWAP 分区并配置启动自动启用**

``` 
mkswap -f /dev/zvol/rpool/swap
echo /dev/zvol/rpool/swap none swap discard 0 0 >> /etc/fstab
echo RESUME=none > /etc/initramfs-tools/conf.d/resume
```

**启用 SWAP**

``` 
swapon -av
```

## 安装完整（full）系统

**从最小化系统升级**

``` 
apt dist-upgrade --yes
```

**安装常用软件包**

``` 
tasksel
```

**关闭 log 压缩**

``` 
# 使用以下脚本

#!/bin/sh
for file in /etc/logrotate.d/* ; do
    if grep -Eq "(^|[^#y])compress" "$file" ; then
        sed -i -r "s/(^|[^#y])(compress)/\1#\2/" "$file"
    fi
done
```

**重启**

``` 
reboot
```

## 清理

重启后如能正确登录账户，检查网络连接正常后，安装成功。

**删除初始安装的快照**

``` 
sudo zfs destroy bpool/BOOT/debian@install
sudo zfs destroy rpool/ROOT/debian@install
```

**关闭 root 密码登录**

``` 
sudo usermod -p '*' root
```

**关闭 root 账户 SSH 登陆**

``` 
vi /etc/ssh/sshd_config

# 找到下面的行并删除 
PermitRootLogin yes

systemctl restart ssh
```

**到这里为止，如果都没有报错，则已经成功地在 ZFS 上安装了 Debian 10 Buster 系统。**

## 创建真用户并加密家目录

**1. 创建一个新用户，本文中称为 `realuser`**

``` 
adduser realuser
```

**2. 保存 `/home/realuser` 中的文件**

``` 
mkdir ~/realuser_home
cp /home/realuser/.* ~/realuser_home
```

**3. 创建 ZFS Mirror pool，称为 `realhome`**

``` 
# 如果磁盘容量不一样，需要加上 `-f`
zpool create realhome mirror /dev/sdY /dev/sdY -f
```

**4. 在 `realhome` pool 中创建 ZFS filesystem**

``` 
zfs create \
    -o encryption=aes-256-gcm \
    -o keysource=passphrase \
    -o keylocation=prompt \
    -o compression=lz4 \
    -o mountpoint=/home/realuser \
    realhome/realuser
```

**5. 修改家目录权属**

``` 
chown realuser:realuser /home/realuser
```

**6. 将保存的文件放到新的家目录**

``` 
cp ~/realuser_home/.* /home/realuser
```

**7. 创建导入 pool、挂载 filesystem、切换用户的脚本**

``` 
#!/bin/bash
zpool import realhome
zfs mount -la
su - realuser
```

### 使用方法

重启服务器后，先通过 SSH 登陆服务器，并使用 `ongo` 用户登录，运行刚才的脚本，输入 filesystem 的密码、 `realuser` 密码后，即可正常使用 `realuser` 用户。

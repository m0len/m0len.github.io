---
layout: post
title: "Debian 10 初探（一）"
comments: true
date: 2019-09-01
description: ""
categories: [OS]
tags: [debian, buster, wifi, apt, ssh]
---

**目录**
* ToC
{:toc}
---

## 配置 `/etc/apt/source.list` 

使用 DVD 版镜像安装之后， `source.list` 文件内会有以下类似仓库：

```sh
deb cdrom:[Debian GNU/Linux 10.0.0 _Buster_ - Official amd64 DVD Binary-1 20190706-10:24]/ buster contrib main
```

这个仓库要先 `comment` 掉， 否则会导致 `apt-get update` 出错。

然后修改以下仓库：

```sh
$ vim /etc/apt/source.list
```

修改为：

```sh
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free

deb http://security.debian.org/debian-security buster/updates main contrib
deb-src http://security.debian.org/debian-security buster/updates main contrib
```

## 修改默认编辑器

```sh
$ update-alternatives --config editor
```

会出现如下选项

```sh
There are 3 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------

* 0            /bin/nano            40        auto mode

  1            /bin/nano            40        manual mode
  2            /usr/bin/vim.basic   30        manual mode
  3            /usr/bin/vim.tiny    15        manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

输入 `2`， 回车。

## 开机后无法直接远程 `SSH` 登录

机器有一张 Wi-Fi 网卡和一个有线网卡，我发现重启后无法直接 `ssh` 连接到服务器（无论通过无线还是有线），需要本地（本机）先登录一下才能 `ssh` 进服务器。下面是一些解决方法。

首先，配置要自动连接的网络：

```sh
$ wpa_passphrase "wifi-ssid" wifi-password > /etc/wpa_supplicant/wpa_supplicant.conf
```

然后，编辑 `/etc/network/interface` :

```sh
$ vim /etc/network/interface
```

添加如下内容，假设网卡为 `wlp2s0` ：

```sh
auto wlp2s0
iface wlp2s0 inet dhcp
pre-up wpa_supplicant -B -iwlp2s0 -c /etc/wpa_supplicant/wpa_supplicant.conf -Dnl80211,wext
post-down killall -q wpa_supplicant
```

重启网卡并连接：

```sh
$ sudo ip link set wlan0 down
$ sudo ip link set wlan0 up
$ sudo wpa_supplicant -B -iwlan0 -c /etc/wpa_supplicant.conf -Dnl80211,wext
$ sudo dhclient wlan0
```

设置 `ssh` 服务端开机自启：

```
$ sudo update-rc.d ssh defaults
```

重启后已经可以直接 `ssh` 连接上机器。


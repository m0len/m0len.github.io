---
layout: post
title: "解决 PVE 虚拟机 CPU 性能差的方案"
comments: true
date: 2019-02-27
description: ""
categories: [VM]
tags: [cpu, vm, virtual, machine, aes, proxmox, pve, openssl, encrypt]
---

**目录**
* ToC
{:toc}
---

## 发现性能问题

最近装了 Proxmox VE，建了虚拟机后跑了一下 AES-NI 的测试：

``` BASH
$ openssl speed -elapsed -evp aes-128-gcm
```

发现虚拟机和宿主机的性能相差几十倍，因为需要虚拟机有强 AES 性能，所以找了一下解决方案，发现是以下问题（摘自 Reddit）：

> Check `/proc/cpuinfo` on the host, and compare it against `/proc/cpuinfo` on the VMs. You'll find the VMs missing essentially all the processor features introduced this century, even though they're supported by your host CPU.

随后检查了一下，发现确实是这样。原来问题出在了建立虚拟机的时候 CPU 类型选择错了。

## 解决方法

非常简单，在 PVE 管理界面中选择修改虚拟机配置：

```sh
Hardware > Processors > Edit > 设置为 host。
```

重新跑分，分数已相差无几，问题解决。

## 参考

1. [AES-NI SSL Performance](https://calomel.org/aesni_ssl_performance.html)
2. [Poor performance on an R710 with Proxmox.2 x L5640 and 64GB of RAM](https://www.reddit.com/r/homelab/comments/6coc6o/poor_performance_on_an_r710_with_proxmox_2_x/)


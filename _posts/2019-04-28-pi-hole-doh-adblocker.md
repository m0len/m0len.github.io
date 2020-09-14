---
layout: post
title: "Pi-Hole 搭配 DoH 加密 DNS 查询以及去广告实践"
comments: true
date: 2019-04-28
description: ""
categories: [Networking]
tags: [lede, pihole, router, gateway, x86, arm, network, doh, cloudflare, adblock]
---

**目录**
* ToC
{:toc}
---

## 安装 Pi-Hole

使用官方安装命令：

```
$ curl -sSL https://install.pi-hole.net | bash
```

之后可以一路回车，安装过程中所有设置都可以安装好之后再改。

## 配置 Pi-Hole

```
# adlists.list
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://mirror1.malwaredomains.com/files/justdomains
http://sysctl.org/cameleon/hosts
https://zeustracker.abuse.ch/blocklist.php?download=domainblocklist
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt
https://hosts-file.net/ad_servers.txt
http://someonewhocares.org/hosts/hosts
https://raw.githubusercontent.com/StevenBlack/hosts/master/data/StevenBlack/hosts
http://www.malwaredomainlist.com/hostslist/hosts.txt
http://adaway.org/hosts.txt
http://winhelp2002.mvps.org/hosts.txt
https://hosts.nfz.moe/full/hosts
file:///root/adblock.hosts #这是自定义 hosts 的格式，Pi-Hole 支持直接读取本地的 hosts 文件
```

* [adlists.list ](https://raw.githubusercontent.com/m0len/m0len.github.io/master/assets/adlists.list)这是我的广告域名列表，放入该文件到 `/etc/pihole/` 中；
* 放入我自己制作的[ adblock.hosts ](https://raw.githubusercontent.com/m0len/m0len.github.io/master/assets/adblock.hosts)列表到根目录；
* 最后，运行 `pihole -g` 命令以更新列表。

## 如何获取自定义去广告列表

这里使用的是 Adblock Plus 中的 Easylist + Easylist China 以及 Malware Domain 和 fanboy-social 的 hosts。

以Easylist + Easylist China为例：

```
# 新建一个 adlists.sh 文件
$ vim adlists.sh

# 写入如下内容
#!/bin/sh
curl -s -L https://easylist-downloads.adblockplus.org/easylistchina+easylist.txt https://easylist-downloads.adblockplus.org/malwaredomains_full.txt https://easylist-downloads.adblockplus.org/fanboy-social.txt > adblock.unsorted
sort -u adblock.unsorted | grep ^\|\|.*\^$ | grep -v \/ > adblock.sorted
sed 's/[\|^]//g' < adblock.sorted > adblock.hosts
rm adblock.unsorted adblock.sorted

# 保存并添加可执行权限
$ chmod +x adlists.sh

# 运行即可得到一个格式正确的 adlists.list
$ ./adlists.sh
```

## 主路由 DHCP 设置

配置好 Pi-Hole 后，只需将主路由 DHCP 设置中的 DNS 服务器设置成 Pi-Hole 的 IP 地址即可。

## 使用 DoH（可选）

#### 什么是 DoH?

> DNS over HTTPS (DoH) is a protocol for performing remote Domain Name System (DNS) resolution via the HTTPS protocol. A goal of the method is to increase user privacy and security by preventing eavesdropping and manipulation of DNS data by man-in-the-middle attacks.
> 
> --Wikipeadia

> 新闻一则：
> 
> [UK ISP group names Mozilla 'Internet Villain' for supporting 'DNS-over-HTTPS'](https://www.zdnet.com/article/uk-isp-group-names-mozilla-internet-villain-for-supporting-dns-over-https/)

总之，DoH 是一个挺好的东西，实现的原理也非常简单。简单理解是先与 DNS 服务器建立一个安全链接，然后再发送 DNS 查询。目前提供 DoH 客户端应用的好像只有 Cloudflare 一家，因此就以 Cloudflare 提供的 `cloudflared` 客户端为例。

#### 下载 `cloudflared` 客户端：

下载页面：[Cloudflare Downloads](https://developers.cloudflare.com/argo-tunnel/downloads/)

| Type | amd64 | x86 | ARMv6 |
| --- | --- | --- | --- |
| Binary  | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.tgz) | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-386.tgz) | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz) |
| .deb | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.deb) | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-386.deb) |  [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.deb)|
| .rpm | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.rpm) | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-386.rpm) | [Download](https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.rpm) |

#### 配置 `cloudflared` 

**说个笑话， Cloudflare 的 `cloudflared` 客户端官方配置教程还没有 Pi-Hole 的教程详细。**

* 创建一个新用户用于 `cloudflared` 

```
$ sudo useradd -s /usr/sbin/nologin -r -M cloudflared 
```

* 创建配置文件

```
$ vim /etc/default/cloudflared
```

* 修改文件权属

```
$ sudo chown cloudflared:cloudflared /etc/default/cloudflared
$ sudo chown cloudflared:cloudflared /usr/local/bin/cloudflared
```

* 创建 `systemd` 脚本以自动启动和后台运行

```
$ vim /lib/systemd/system/cloudflared.service
```

* 写入如下内容

```
[Unit]
Description=cloudflared DNS over HTTPS proxy
After=syslog.target network-online.target

[Service]
Type=simple
User=cloudflared
EnvironmentFile=/etc/default/cloudflared
ExecStart=/usr/local/bin/cloudflared proxy-dns --port 5053
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

* 自启及运行

```sh
$ sudo systemctl enable cloudflared
$ sudo systemctl start cloudflared
$ sudo systemctl status cloudflared
```

* 测试

```sh
$ dig @127.0.0.1 -p 5053 google.com
```

出现以下输出说明正常运行

```sh
; <<>> DiG 9.10.3-P4-Ubuntu <<>> @127.0.0.1 -p 5053 google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65181
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1536
;; QUESTION SECTION:
;google.com.            IN  A

;; ANSWER SECTION:
google.com.     299 IN  A   243.65.127.221

;; Query time: 3 msec
;; SERVER: 127.0.0.1#5053(127.0.0.1)
;; MSG SIZE  rcvd: 65
```

* 修改 Pi-Hole 设置

在 Pi-Hole 管理页面中， `Settings` - `DNS` 中取消勾选 Google，勾选右方自定义并填写上游 DNS 服务器： `127.0.0.1#5053` ，点击保存。

## 参考链接

1. [Pi-Hole WiKi](https://github.com/pi-hole/pi-hole/#one-step-automated-install)
2. [Adblock Plus subscriptions](https://adblockplus.org/en/subscriptions)
3. [UK ISP group names Mozilla 'Internet Villain' for supporting 'DNS-over-HTTPS'](https://www.zdnet.com/article/uk-isp-group-names-mozilla-internet-villain-for-supporting-dns-over-https/)
4. [Running a DNS over HTTPS Client](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/)
5. [Configuring DNS-Over-HTTPS on Pi-hole](https://docs.pi-hole.net/guides/dns-over-https/)


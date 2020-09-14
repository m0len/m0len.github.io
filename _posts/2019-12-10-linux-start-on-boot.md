---
layout: post
title: "Linux 开机启动/运行脚本的方法"
comments: true
date: 2019-12-10
description: ""
categories: [OS]
tags: [linux, chkconfig, systemd]
---

**目录**
* ToC
{:toc}
---

> 近期在配置一个服务器监控软件，软件需要开机启动，所以研究一下自启脚本。

## 方法一：`systemd`

> 该方法在支持 `systemd` 的系统上都可以正常工作。

### 创建一个可执行脚本，内容是需要启动运行的命令

```sh
$ vim startup.sh
```

内容如下：

```sh
#!/bin/sh

command ...
command ...
command ...
```

添加可执行权限：

```sh
$ chmod +x startup.sh
```

### 创建 `systemd service`

```sh
$ vim /usr/local/lib/systemd/system/mystartup.service
```

内容如下：

```sh
[Unit]
Description=Startup Script on Boot

[Service]
ExecStart=/path/to/startup.sh

[Install]
WantedBy=multi-user.target
```

### 刷新 `systemd daemon` 并设置启动时运行

```sh
$ systemctl daemon-reload
$ systemctl enable mystartup.service
```

### 测试能否运行

```sh
$ systemctl start mystartup.service
```


## 方法二：`init.d`

> 该方法在某些较新的系统上无效。

### 添加启动脚本 `/etc/init.d/xxx`

本文用的是在 `/etc/init.d` 文件夹下创建启动脚本的方法。

脚本内容如下：

```sh
#!/bin/sh
#chkconfig: 2345 80 90
#description:auto_run

command1
command2
```

以下摘自[此文](https://blog.51cto.com/17610376/322834)：
> 第一行，告诉系统使用的 shell，所有 shell 脚本都是这样。
> 第二、三行，`chkconfig` 后面有三个参数 2345，80 和 90 告诉 `chkconfig` 程序，需要在 `rc2.d` ~ `rc5.d` 目录下，创建名字为 `S80auto_run` 的文件连接，连接到 `/etc/rc.d/init.d` 目录下的的 `auto_run` 脚本。第一个字符是 S，系统在启动的时候，运行脚本 `auto_run`，就会添加一个 `start` 参数，告诉脚本，现在是启动模式。同时在 `rc0.d` 和 `rc6.d` 目录下，创建名字为 `K90auto_run` 的 文件连接，第一个字符为 K ，在关闭系统的时候，会运行 `auto_run`，添加一个 `stop` ，告诉脚本，现在是关闭模式。

### 设置自启服务

```sh
chkconfig xxx on
```

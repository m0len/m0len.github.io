---
layout: post
title: "Docker 初探（持续更新中）"
comments: true
date: 2019-09-15
description: ""
categories: [Container]
tags: [docker, container, virtual, dockerfile, compose, alpine]
---

**目录**
* ToC
{:toc}
---

| 版本历史 |
| --- | --- |
| 2019/09/15 | 开篇 |


> 新手可能一开始会混淆 `Dockerfile` 和 `docker-compose` 。两者区别在于 `Dockerfile` 是用于编写镜像 `image` ， 而 `docker-compose` 是用于运行镜像 `image` 。一个私以为贴切的比喻：把 `docker` 比作一杯鸡尾酒，写 `Dockerfile` 就像在制作鸡尾酒中的每一款酒，而写 `docker-compose.yml` 就像是调酒的过程。

## Dockerfile

* `Dockerfile` 中可以使用多个 `stage` ，最常见的场景是第一个 `stage` 用于编译容器内的程序 `build` ，第二个 `stage` 用于真正运行该程序的环境，这样可以减小 `image` 体积。不同 `stage` 之间使用 `FROM` 区分开。

```sh
# 第一层
FROM ubuntu:latest AS Builder
...
...
...
# 第二层
FROM alpine:latest
...
...
...
```

* `Dockerfile` 中每一个关键词都会创建一个层 `layer` ，层数会影响最终 `image` 的体积，所以在实现功能的前提下尽量少增加层。比如同一个 `stage` 中的 `RUN` 命令可以多条命令写成一句（使用 `&&` 分割）。
* `ENV` 用于定义环境变量，作用类似命令行界面中的 `export XXX=value` ，对容器内的编译起作用。  *如：编译 GO 程序时，定义环境变量 `ENV GO111MODULE=on` *
* `ENTRYPOINT` 描述了一个 `image` 的初始命令，**必须要有**且不会被运行时的命令覆盖。相对地，`CMD` 作为 `ENTRYPOINT` 的补充，**可有可无**且可被运行容器时传递的命令覆盖。两者的搭配可见[官方文档](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)
* `Dockerfile` 不支持占位符（如\$1、\$2）

## Docker-compose

* `EXPOSE` 和 `PORTS` 的区别在于前者会被随机安排一个端口映射，而后者需要用户自定义映射或者不映射。
* Docker 用于获取和控制容器状态的 `docker.socks` 。如果需要控制容器运行，如使用容器管理软件或需要实时修改容器，可以挂载 `/var/run/docker.socks:/var/run/docker.socks` 


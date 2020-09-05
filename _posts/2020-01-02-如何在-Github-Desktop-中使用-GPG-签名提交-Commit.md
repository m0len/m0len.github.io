---
layout: post
title: "如何在 Github Desktop 中使用 GPG 签名提交 Commit"
comments: true
date: 2020-01-02
description: ""
keywords: "github, gpg, commit, desktop"
categories: [Security]
---

> 2020的第一篇。新的一年，祝好。

## 生成密钥对并添加到 Github

这一步非常简单，可以参考[官方教程](https://help.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key)，以下提供简略步骤。

### 1

Windows 下可以使用 Git 图形客户端（[Git Bash](https://git-scm.com/downloads)）中的 `gpg` 命令，Linux 和 Mac 下可以用 Terminal 安装 `gpg` 工具。

Windows 下生成密钥对的命令如下：

``` BASH
$ gpg --full-generate-key
```

这里有几个需要注意的地方：

1. Github 的密钥需要使用 `4096` bits；
2. user name 填写 Github 的登录名；
3. 如果开了 private mail 的话 mail 填写 Github 提供的那个 noreply mail 地址。

### 2

列出所有的密钥，并获取密钥 ID

``` BASH
$ gpg --list-secret-keys --keyid-format LONG

----------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot 
ssb   4096R/42B317FD4BA89E7A 2016-03-10
```

以上例子中， `3AA5C34371567BD2` 就是所需要的密钥 ID。

使用 ASCII 格式导出公钥：

``` BASH
$ gpg --armor --export 3AA5C34371567BD2
```

将输出中的 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 和 `-----END PGP PUBLIC KEY BLOCK-----` 和他们之间的所有内容复制到 Github 设置中，添加 GPG 密钥完成。

## 告知 Git 要使用 GPG 签名

可以参考[这篇](https://gist.github.com/xavierfoucrier/c156027fcc6ae23bcee1204199f177da)，以下提供简略步骤。

### 1

打开 `.gitconfig` 文件

``` BASH
# 方法一
$ git config --global --edit
# 方法二
$ vim C:\Users\xxx\.gitconfig
```

### 2

添加以下内容

``` INI
[user]
  name = YOUR_GITHUB_NAME  #用户名
  email = YOUR_GITHUB_EMAIL  #邮箱
  signingkey = YOUR_SIGNING_KEY  #GPG密钥ID
[gpg]
  program = GPG_BINARY_PATH  #GPG程序路径
[commit]
  gpgsign = true  #对Commit启用GPG签名
```

这里有一个需要注意的地方：

1. Windows 下的 GPG 程序路径需要这样的格式： `C:\\Program Files\\Git\\usr\\bin\\gpg.exe` （注意双反斜杠 `\` ），否则会出现配置文件错误 `fatal: error in line xx` 。

### 3

**!!! 重点注意：Windows 和 Mac 系统下需要安装一个 GPG 图形客户端才能正常使用 GPG 签名。否则在 `Commit` 时会出现类似以下的错误，这是缺少输入密钥密码的地方所导致的。**

``` BASH
Commit failed - exit code 128 received, with output: 'error: gpg failed to sign the data
fatal: failed to write commit object'
```

Mac 下推荐安装 [Mac GPG](https://gpgtools.org/)，Windows 下推荐安装 [Gpg4win](https://gpg4win.org/download.html)。以上两个都是 [GnuPG 官网](https://gnupg.org/download/index.html)推荐的 GPG 管理工具。

完成以上三个步骤之后回到 Github Desktop 测试 `Commit` ，会弹出一个对话框要求输入 GPG 密钥密码，说明配置成功。

**- - 全文终 - -**


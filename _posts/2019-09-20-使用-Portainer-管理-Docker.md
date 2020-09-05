---
layout: post
title: "使用 Portainer 管理 Docker"
comments: true
date: 2019-09-20
description: ""
keywords: "docker, portainer, gui, linux"
categories: [Container]
---

> 本文以 CentOS 7 系统为例，其他发行版可能略有不同

## 修改 Docker 启动参数

使用编辑器打开 docker 的 systemd 文件：

```sh
$ vim /usr/lib/systemd/system/docker.service
```

找到 `[Service]` 这段，把 `ExecStart` 后面的内容修改如下：

```sh
[Service]
......
......
ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock
......
......
```

完成后保存。

## 生成自签名证书以使用 SSL 安全链接

这里使用了 **anyesu**（[GitHub](https://github.com/anyesu)，[简书](https://www.jianshu.com/u/c5327915649c)） 的[ TLS 证书一键脚本](https://www.jianshu.com/p/7ba1a93e6de4)。省了很多功夫，特此感谢。

```sh
#!/bin/bash
# @author: anyesu

if [ $# != 1 ] ; then 
echo "USAGE: $0 [HOST_IP]" 
exit 1; 
fi 

#============================================#
#    下面为证书密钥及相关信息配置，注意修改     #
#============================================#
PASSWORD="Yeh377&*8663yeh[3*&*3hi"
COUNTRY=US
PROVINCE=CA
CITY=LA
ORGANIZATION=dtcokrInc
GROUP=admin
NAME=dtcokr
HOST=$1
SUBJ="/C=$COUNTRY/ST=$PROVINCE/L=$CITY/O=$ORGANIZATION/OU=$GROUP/CN=$HOST"

echo "your host is: $1"

# 1.生成根证书RSA私钥，PASSWORD作为私钥文件的密码
openssl genrsa -passout pass:$PASSWORD -aes256 -out ca-key.pem 4096

# 2.用根证书RSA私钥生成自签名的根证书
openssl req -passin pass:$PASSWORD -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem -subj $SUBJ

#============================================#
#          用根证书签发server端证书           #
#============================================#

# 3.生成服务端私钥
openssl genrsa -out server-key.pem 4096

# 4.生成服务端证书请求文件
openssl req -new -sha256 -key server-key.pem -out server.csr -subj "/CN=$HOST"

# 5.使tls连接能通过ip地址方式，绑定IP
echo subjectAltName = IP:127.0.0.1,IP:$HOST > extfile.cnf

# 6.使用根证书签发服务端证书
openssl x509 -passin pass:$PASSWORD -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

#============================================#
#          用根证书签发client端证书           #
#============================================#

# 7.生成客户端私钥
openssl genrsa -out key.pem 4096

# 8.生成客户端证书请求文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

# 9.客户端证书配置文件
echo extendedKeyUsage = clientAuth > extfile.cnf

# 10.使用根证书签发客户端证书
openssl x509 -passin pass:$PASSWORD -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf

#============================================#
#                    清理                    #
#============================================#
# 删除中间文件
rm -f client.csr server.csr ca.srl extfile.cnf

# 转移目录
mkdir client server
cp {ca,cert,key}.pem client
cp {ca,server-cert,server-key}.pem server
rm {cert,key,server-cert,server-key}.pem

# 设置私钥权限为只读
chmod -f 0400 ca-key.pem server/server-key.pem client/key.pem
```

新建一个 `tls.sh` 文件，写入以上内容并保存，给 `tls.sh` 添加可执行权限并运行，即可自动在当前目录生成**根证书，服务端证书和客户端证书**。

```sh
#新建 tls.sh 文件并粘贴以上内容
$ vim tls.sh

#添加可执行权限
$ chmod +x tls.sh

#运行脚本，将 xxx.xxx.xxx.xxx 替换成本机 IP
$ ./tls.sh xxx.xxx.xxx.xxx

#列出生成的证书文件
$ ls ./*
./client:
ca.pem  client-cert.pem  client-key.pem

./server:
ca.pem  server-cert.pem  server-key.pem 
```

生成证书后，将服务端证书移动到 `/etc/docker/` 目录下。

```sh
$ mv ./server/* /etc/docker/
```

## 在防火墙开启 2376 端口允许 SSL 连接

```sh
$ iptables -A INPUT -p tcp -m tcp --dport 2376 -j ACCEPT
```

##### *可选：仅允许指定 IP 地址访问 2376 端口

```sh
#将 xxx.xxx.xxx.xxx 换成管理机的 IP 地址
$ iptables -A INPUT -s xxx.xxx.xxx.xxx -p tcp -m tcp --dport 2376 -j ACCEPT
```

## 配置 Docker Daemon

由于 Docker 默认是不开启远程访问的，所以需要配置一下 Daemon 以启用它。方法是在 `/etc/docker/` 目录下新建一个 `daemon.json` 配置文件。

```sh
#新建 daemon.json
$ vim /etc/docker/daemon.json

#写入以下内容并保存
{
    "debug": true,
    "tls": true,
    "tlscacert": "/etc/docker/ca.pem",
    "tlscert": "/etc/docker/server-cert.pem",
    "tlskey": "/etc/docker/server-key.pem",
    "hosts": ["tcp://0.0.0.0:2376", "fd://", "unix:///var/run/docker.sock"]
}
```

## 在管理机上运行 Portainer

首先贴一个官网地址：[Portainer.io](https://www.portainer.io/)

官网推荐使用 Docker 来部署，我也是这样做的，这是最简便的方式：

```sh
#注意挂载 docker.sock，这是控制本地 Docker Container 的一个接口
$ docker run -d -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

通过反代或者本机直接访问 `http://localhost:9000` 就可以登陆 Protainer 图形管理界面。

#### 配置 Portainer 并添加客户端节点

以下是添加节点的步骤：

![img](https://github.com/m0len/m0len.github.io/raw/master/img/portainer-left.png)

在左边栏点选 Endpoints

![img](https://github.com/m0len/m0len.github.io/raw/master/img/portainer-new-endpoint.png)

这里点选 Add endpoint

![img](https://github.com/m0len/m0len.github.io/raw/master/img/portainer-configure-endpoint.png)

* 上方选择 **Docker** 默认不用管；
* **Name** 填写自己喜欢的；
* **Endpoint URL** 填写客户端的 IP 地址（或域名）加上端口（就是刚才设置的端口 2376），如：192.168.1.2:2376；
* **Public IP** 可填可不填，填的话就和上面的 Endpoint URL 一致；
* **TLS** 需要启用
    - 直接默认选择第一个 **TLS with server and client verification**
    - **TLS CA certificate** 选择刚刚生成的 `ca.pem` 
    - **TLS certificate** 选择刚刚生成的 `client-cert.pem` 
    - **TLS key** 选择刚刚生成的 `client-key.pem` 
* 都填妥之后点击最下方 **Add endpoint**，完成添加。

添加节点之后可以使用 portainer 直接图形管理服务端的 Docker Container，还是非常方便的，不需要输命令，也可以批量管理多个节点多个容器。

**- - 全文终 - -**


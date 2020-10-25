---
layout: post
title: "使用 Grafana 监控 Asus 梅林固件路由器工况"
comments: true
date: 2020-10-25
description: ""
categories: [Networking]
tags: [grafana, asus, merlin, python, crawling, crawler]
---

**目录**
* ToC
{:toc}
---

![asusMonitor](https://github.com/m0len/m0len.github.io/raw/main/img/asusMonitor.png)

## Why

所使用的网络环境是一台 Asus RT-AC1900P 作为硬路由使用，兼职作为无线 AP，所以在网络监控和管理上不如软路由般灵活，因此只能找些别的方法来监控网络使用情况，顺便也可以监控路由器健康状况。

首先了解到的是使用 SNMP 协议，这是梅林插件本身支持的，提供了一些查询路由器工况的接口。根据[这篇文章](https://community.home-assistant.io/t/advanced-snmp-monitoring-part-one-asuswrt-routers-merlin-build/68984)所说，可以查询到包括温度、客户端数量、内存、CPU 空闲等数据，但是对带宽数据依然无解。如果只需要监控路由器健康状况的话，以上数据其实也基本足够了，但是我的需求是加上网络使用情况，所以需要另辟蹊径。

在路由器的管理页面上，有我所需要的所有数据，于是希望用爬虫的方式从这里提取数据，经过处理之后放进 MySQL 数据库中作为数据源，然后用 Grafana 面板展示数据。经过一番研究和完善，发现这算是最容易的方法了。

**完整工作流：**
``` bash
爬虫提取数据 -> 存放在 MySQL -> Grafana 展示图表
```

## 数据篇

在路由器管理页打开浏览器控制台就能看到一堆定时的 XHR，通过观察很容易就能找到查询数据的几个请求 URL，分别是：
1. `http://ip/update.cgi` ——查询网速（WAN、WiFi、LAN）
2. `http://ip/ajax_coretmp.asp` ——查询温度（2.4G芯片、5G芯片、CPU）
3. `http://ip/cpu_ram_status.xml` ——查询 CPU 和内存使用

以上三个 URL 都是通过读取 Cookies 来认证，然后在 Cookie 里面可以找到一个键值对 `'asus_token': ''`，值是一串数字。那么如何获得这串数字呢，再登录一次页面，观察登录请求就会发现另一个 URL `http://ip/login.cgi`，POST 请求里面有一串 Base64 编码的字符串，通过解码就可以发现是 `user:password` 的格式。到此，整个登录的流程已经清楚了。

通过以上 URL，就可以用一个脚本定时获取所需的所有数据，写入 MySQL 后端存储。

**示例代码：**
``` python
import requests
import regex as re
import time
import xml.etree.ElementTree as ET
import urllib
import base64

## Your router's address (format: http://ip_address or https://ip_address)
asusAddr = '' # eg. http://192.168.1.1
## Your Asus router's management web page login user and password
asusUser = ''
asusPasswd = ''

authURL = '{}/login.cgi'.format(asusAddr)
speedURL = '{}/update.cgi'.format(asusAddr)
tempURL = '{}/ajax_coretmp.asp'.format(asusAddr)
statusURL = '{}/cpu_ram_status.xml'.format(asusAddr)

asusIP = re.search(r'((?<=https://)|(?<=http://))\d+\.\d+\.\d+\.\d+', asusAddr)[0]
asusAuth = base64.b64encode('{}:{}'.format(asusUser, asusPasswd).encode())
asusAuthEncode = urllib.parse.quote(asusAuth)

sessions = requests.Session()

authPayload = 'group_id=&action_mode=&action_script=&action_wait=5&current_page=Main_Login.asp&next_page=index.asp&login_authorization={}'.format(asusAuthEncode)
authHeaders = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36',
  'Content-Type': 'application/x-www-form-urlencoded',
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
  'Accept-Encoding': 'gzip, deflate, br',
  'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7',
  'Cache-Control': 'max-age=0',
  'Connection': 'keep-alive',
  'Content-Length': '154',
  'host': '{}'.format(asusIP),
  'referer': '{}/Main_Login.asp'.format(asusAddr)
}

dataPayload = "output=netdev&_http_id=TIDe855a6487043d70a"
dataHeaders = {
  'Cookie': '',
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.3',
  'Content-Type': 'text/plain',
  'Referer': '{}/device-map/router_status.asp'.format(asusAddr)
}

# 获取 token
def getAuth():
    global token
    # ignore tls warning
    requests.packages.urllib3.disable_warnings()
    
    sessions.post(authURL, headers=authHeaders, data = authPayload, verify=False)
    token = sessions.cookies.get_dict()['asus_token']
    # print(token)
    logging.info('New Token: {}'.format(token))
    return token

# 获取数据
def getData(token):
    # ignore tls warning
    requests.packages.urllib3.disable_warnings()
    
    # cookie
    dataHeaders['Cookie'] = 'asus_token=' + token + 'clickedItem_tab=0'
    
    # get speed and cpu ram status
    speedResponse1 = sessions.post(speedURL, headers=dataHeaders, data=dataPayload, verify=False)
    statusResponse1 = sessions.get(statusURL, headers=dataHeaders, verify=False)
    time1 = time.time()
    time.sleep(1)
    speedResponse2 = sessions.post(speedURL, headers=dataHeaders, data=dataPayload, verify=False)
    statusResponse2 = sessions.get(statusURL, headers=dataHeaders, verify=False)
    time2 = time.time()
    
    # get coretemp
    tempResponse = sessions.get(tempURL, headers=dataHeaders, verify=False)

    # speed result
    speedResult1 = speedResponse1.text
    speedResult2 = speedResponse2.text
    
    # coretemp result
    tempResult = tempResponse.text
    
    # cpu ram status result
    statusResult1 = ET.fromstring(statusResponse1.text)
    statusResult2 = ET.fromstring(statusResponse2.text)
    ...
```
稍微处理一下数据即可。


## 数据库篇

数据获取到了，怎么存储也是一个问题，但是这个问题简单。

**Grafana 官方支持以下数据源：**
* AWS CloudWatch
* Azure Monitor
* Elasticsearch
* Google Cloud Monitoring
* Graphite
* InfluxDB
* Loki
* Microsoft SQL Server (MSSQL)
* MySQL
* OpenTSDB
* PostgreSQL
* Prometheus
* Jaeger
* Zipkin
* Tempo
* Testdata

可以看到基本上都是各种数据库，所以这里选一个自己熟悉的就可以了，这里我选择了 MySQL。为了方便起见，直接用 Docker 搭建。

**Docker Compose 配置：**
``` yaml
version: "3"

services:
  mysql:
    image: mysql:5.7.31
    container_name: mysql
    network_mode: host
    volumes:
      - /root/mysql-data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
    restart: always
```

**创建数据库：**
``` sql
mysql> CREATE DATABASE db_name;
```

**创建数据表：**
共 14 种数据，加 1 个时间戳。
``` sql
mysql> CREATE TABLE table_name (
          internetRXSpeed FLOAT, 
          internetTXSpeed FLOAT, 
          wiredRXSpeed FLOAT, 
          wiredTXSpeed FLOAT, 
          wireless2gRXSpeed FLOAT, 
          wireless2gTXSpeed FLOAT, 
          wireless5gRXSpeed FLOAT, 
          wireless5gTXSpeed FLOAT, 
          coretemp2g INT, 
          coretemp5g INT, 
          coretempCPU INT, 
          cpu1Percentage FLOAT, 
          cpu2Percentage FLOAT, 
          ramUsage INT, 
          timeStamp TIMESTAMP NOT NULL PRIMARY KEY);
```

## 数据展示篇

Grafana 是一个常用的数据可视化工具，提供多种面板和相对自由而简单的设置，还能实时展示。同样地，为了方便，使用了 Docker 搭建。

**Docker Compose 配置：**
Grafana Docker 的配置推荐使用环境变量来设置，而不是挂载配置文件并修改。
``` yaml
version: "3"

services:
  grafana:
    image: grafana/grafana
    container_name: grafana
    user: "0"
    restart: always
    network_mode: host
    volumes:
      - /root/grafana-data:/var/lib/grafana
    environment:
      - GF_DASHBOARDS_MIN_REFRESH_INTERVAL=1s
      - GF_AUTH_ANONYMOUS_ENABLED=true
```

至此，通过一些设置，就可以调出自己需要的数据展示方式。

需要注意的是，Grafana 渲染和数据库查询需要占一些资源。我这里使用安装在一台 Intel J3455 CPU 的迷你主机上的虚拟机来作为服务器，配置是单核、1GB 内存。按图示面板，时间范围为 30 分钟，查询间隔 1 秒的使用场景下，平均 CPU 占用约 15%。时间范围设置为 6 小时的时候，就可能会出现无法渲染图表的情况。
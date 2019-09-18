---
layout: title
title: STUN和TURN服务器搭建
date: 2019-03-02 23:35:49
tags: [WebRTC]
categories: [技术分析]
---

使用CoTurn自建STUN和TURN服务。

<!--- more --->

### 下载 [CoTurn](https://github.com/coturn/coturn/blob/master/INSTALL)

下载指定版本的CoTurn并解压

```
wegt https://coturn.net/turnserver/v4.5.0.8/turnserver-4.5.0.8.tar.gz
tar -zxvf turnserver-4.5.0.8.tar.gz
```

### 安装

安装编译所需依赖

```
apt-get update
apt-get install -y libssl-dev libevent-dev libpq-dev mysql-client libmysqlclient-dev libhiredis-dev openssl
apt-get install gdebi-core -y
apt-get install sqlite libsqlite3-dev -y
```

编译安装CoTurn

```
cd turnserver-4.5.0.8
./configure
make
make install
```

查看是否安装成功

```
which turnserver
```

### 配置

添加用户

```
turnadmin -a –u 用户名 -r 域名 -p 密码
```

例如：添加用户sccs，密码为123456，域名fongda.com（域名可以随意填写）

```
turnadmin -a -u sccs -r fongda.com -p 123456
```

输入`turnadmin -l`检查是否添加用户成功

### 启动

前台启动

```
turnserver -L 公网IP -a -f -r 域名
```

或者在后台运行

```
$ turnserver -L 公网IP -o -a -f -r 域名
```

### 验证

使用webrtc官方提供的网址进行验证：

https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

输入要验证的STUN和TURN服务器

![image](https://wx3.sinaimg.cn/large/005PPQ5Ily1g0ovehi2xbj31gs0iidhl.jpg)

出现Done时表示服务正常运行

![image](https://wx2.sinaimg.cn/large/005PPQ5Ily1g0ovfxi3yzj31ie09maay.jpg)


---
layout: title
title: WebRTC与NAT穿透
date: 2019-03-02 23:53:08
tags: [WebRTC]
categories: [技术分析]
---

WebRTC支持点对点通信（P2P），但是其建立通讯仍需要两类服务器。

- 信令服务器：用于客户端交换元信息以协调通讯。
- STUN/TURN服务器：应对网络地址转换器（NAT）和防火墙。

<!--- more --->

> 以下客户端指进行通话的用户

### 信令服务器

信令就是协调通讯的过程，为了建立一个webRTC的通讯过程，客户端需要交换如下信息：

- 用于打开或关闭通信的会话控制消息。
- 错误消息。
- 媒体元数据，例如编解码器和编解码器设置，带宽和媒体类型。
- 密钥数据，用于建立安全连接。
- 网络数据，例如客户端的外网IP地址和端口。

### STUN/TURN服务器

对于建立连接所需的元数据信息，WebRTC通过信令服务器进行转发。但是对于实际的媒体和数据流，一旦建立会话，首先尝试直接连接客户端，即客户端点对点通信。也就是说，每个客户端都有一个唯一的IP地址，他能用来和其他客户端进行直接通讯。

![没有NAT和防火墙的理想情况](https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/p2p.png)

<center><i>没有NAT和防火墙的理想情况</i></center>

但是现实中，大多数设备都位于一层或多层NAT之后，有些设备具有阻止某些端口和协议的防病毒软件，而且许多设备都在代理和企业防火墙之后。 防火墙和NAT也可以由相同的设备实现，例如家庭wifi路由器。

![](https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/nat.png)

<center><i>现实世界</i></center>

WebRTC可以使用ICE框架来克服现实网络的复杂性。 在连接过程中，ICE将试图找到最佳的连接路径，通过尝试所有可能的选项,然后选择最合适的方案。ICE首先尝试使用从设备的操作系统和网卡获得的主机地址建立连接；如果失败（设备位于NAT后），ICE将使用STUN服务器获取设备外网地址；如果失败，则使用TURN服务器作为中继转发流量（媒体和数据流）。

也就是说：

- STUN服务器用于获取设备的外网地址。
- TURN服务器用于转发流量，在P2P连接失败的情况下发挥作用。

STUN和或TURN服务器的URL（可选）由iceServers配置对象中的WebRTC应用程序指定，该对象是RTCPeerConnection构造函数的第一个参数：

```javascript
{
  'iceServers': [
    {
      'urls': 'stun:stun.l.google.com:19302'
    },
    {
      'urls': 'turn:192.158.29.39:3478?transport=udp',
      'credential': 'JZEOEt2V3Qb0y27GRntt2u2PAYA=',
      'username': '28224511:1379330808'
    },
    {
      'urls': 'turn:192.158.29.39:3478?transport=tcp',
      'credential': 'JZEOEt2V3Qb0y27GRntt2u2PAYA=',
      'username': '28224511:1379330808'
    }
  ]
}
```

#### STUN服务器

NAT为设备提供IP地址，以便在内网中使用，但此地址不能在外部使用。如果没有公共地址，WebRTC对等方就无法进行通信。为了解决这个问题，WebRTC使用STUN服务器。

STUN服务器位于**公网**上，其工作非常简单：检查传入请求的IP:port，并将该地址作为响应发回。也就是说，应用程序使用STUN服务器从外网是脚下发现其IP:Port。此过程使WebRTC客户端获取自己的公网地址，然后通过信令机制将其传递给另一个苦护短，以便建立P2P连接。

**STUN服务器的工作非常简单，因此可以处理大量请求**，这也是有很多免费的STUN服务器提供的原因。

> 根据webrtcstats.com，WebRTC使用STUN成功建立连接的概率为：86％

![](https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/stun.png)

<center><i>使用STUN服务器获取公网地址</i></center>

#### TURN服务器

在P2P连接失败后，WebRTC会利用TURN服务器作为中继进行通信。需要强调的是：**TURN在不同客户端之间中转的是音频/视频/数据流，而不是信令数据。**

TURN服务器具有公网地址，因此即使客户端位于防火墙或代理之后，也可以与其进行联系。 TURN服务器需要中转大量的视频音频等数据，因此相对STUN服务器，它们需要消耗大量带宽。

![](https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/turn.png)

<center><i>完整的框架：STUN、TURN、SIGNALING</i></center>

在上图中，若使用STUN无法建立连接，将会使用TURN进行中继。

### 附录：NAT分类与STUN/TURN

#### 基础概念

一个IP连接由两个IP组成的连接“四元组”决定：

- (Source IP address, source port number)
- (Destination IP address, destination port number)

即（源地址，源端口）和（目标地址，目标端口），可视化如下：

```
(SRC_ADDR, SRC_PORT) -> (DST_ADDR, DST_PORT)
```

> NOTE
>
> - `->` 表示连接的方向
> - `(ADDR, PORT)` 表示一个（IP地址，IP端口）元组
> - `(SRC_ADDR, SRC_PORT)  ` 表示发起连接的机器的IP
> - `(DST_ADDR, DST_PORT)` 表示接收连接的机器的IP

NAT(Network Address Translation) 是一种广泛应用的解决IP 短缺的有效方法， NAT 将内网地址转和端口号换成合法的公网地址和端口号，建立一个会话，与公网主机进行通信。可视化如下：

NAT分为两大类：锥型和对称型，而锥型又分为三种类型，分类如下：

- 锥形 cone NAT
  - 全锥形 Full Cone NAT
  - 地址受限锥形 Address-Restricted Cone NAT
  - 端口受限锥形 Port-Restricted Cone NAT
- 对称形 Symmetric NAT

#### 全锥形 Full Cone NAT

全锥NAT 把所有来自相同内部IP地址和端口的请求映射到相同的外部IP 地址和端口。任何一个外部主机均可通过该映射发送数据包到该内部主机，可视化如下：

```
    {NAT internal side}  |    {NAT external side}  |  {Remote machine}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (   *    ,    *    ) ]
```

> `*`表示所有值都可以使用，任何一个外部主机都可以通过该映射建立连接。

#### 地址受限锥形 Address-Restricted Cone NAT

地址受限锥形NAT 把所有来自相同内部IP 地址和端口的请求映射到相同的外部IP 地址和端口。但是, 和全锥NAT 不同的是：只有当内部主机先给外部主机发送数据包, 该外部主机才能向该内部主机发送数据包，可视化如下：

```
    {NAT internal side}  |    {NAT external side}  |  {Remote machine}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (REM_ADDR,    *    ) ]
```

> 在第二步中，REM_ADDR必须和第一步中相同，但是REM_PORT没有限制。

#### 端口受限锥形 Port-Restricted Cone NAT

端口受限锥形NAT 与地址受限锥形NAT 类似, 只是多了端口号的限制, 即只有内部主机先向外部地址和端口号元组发送数据包, 该外部主机才能使用特定的端口号向内部主机发送数据包，可视化如下：

```
    {NAT internal side}  |    {NAT external side}  |  {Remote machine}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, INT_PORT) -> (REM_ADDR, REM_PORT) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, INT_PORT) <- (REM_ADDR, REM_PORT) ]
```

> 在第二步中，REM_ADDR和REM_PORT都要和第一步相同。

#### 对称形 Symmetric NAT

对称形NAT 与上述3 种锥类都不同，当同一内部主机使用相同的端口与不同地址的外部主机进行通信时, 对称NAT 会重新建立一个Session ，为这个Session 分配新的外部端口号，或许还会改变IP 地址。 这意味着从同一本地端口到两个不同远程计算机的两个连续连接将具有两个不同的外部端口，即使是同一内部主机，可视化如下：

```
    {NAT internal side}  |    {NAT external side}  |  {Remote machine}
                         |                         |
1. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, EXT_PORT1) -> (REM_ADDR, REM_PORT1) ]
2. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, EXT_PORT1) <- (REM_ADDR, REM_PORT1) ]
...
3. (INT_ADDR, INT_PORT) => [ (EXT_ADDR, EXT_PORT2) -> (REM_ADDR, REM_PORT2) ]
4. (INT_ADDR, INT_PORT) <= [ (EXT_ADDR, EXT_PORT2) <- (REM_ADDR, REM_PORT2) ]
```

> 在第一步中建立连接，分配的外部端口号为EXT_PORT1
>
> 在第三步中建立新的连接，分配的外部端口号为EXT_PORT2，**外部端口号发生变化**。

#### STUN/TURN与NAT的关系

假设客户端A和客户端B需要建立P2P连接，在锥形NAT中，由于`(EXT_ADDR, EXT_PORT)`是不会发生变化的，所以我们可以通过STUN获取该客户段A和B的外部IP和端口，然后让两者建立连接。

在对称形NAT中，由于`(EXT_ADDR, EXT_PORT)`中的端口会发生变化，当客户端A和B与STUN服务器建立连接，获取外部IP和端口后。两者利用该外部IP和端口建立P2P连接，对称NAT会为A和B分配新的外部端口，因此无法建立连接。所以，对于对称形NAT，我们需要通过TURN服务器进行信息转发。
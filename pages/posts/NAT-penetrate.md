---
title: '如何更方便地访问 NAT 后面的 HTTP Server？'
date: 2017/07/07
tag: socket network
author: Andy
---
这篇文章整整拖了 5 个月，生成的 Timestamp 原来是 2 月 7 的，现在都 7 月了，拖延症很严重。

有个需要本来是一个很简单的 C/S 模型，机器人 HTTP Server 在本地局域网中提供给客户端（Web，桌面，安卓/iOS）访问。客户端连上机器人发起的本地局域网，然后通过本地局域网的 IP 地址，访问机器人上的 HTTP Server，控制机器人。

但是后面有一个需求是，当机器人插上一个 3/4 G 网卡上网的时候，要复用机器人上的 HTTP Server 远程控制机器人。那问题来了，该如何访问一个 NAT 后面的 HTTP Server 呢？

<!--more-->

原来在做这个项目的时候调研过几个方案。

1. 用 SSH 反向隧道，把本地的 HTTP Server port 映射到公网上的某台服务器上。但是弊端很多，一个机器人 Server Port 对应一个公网 Server Port，机器人上会有很多不同后端工程师写的 HTTP Server Port，而且有几百个机器人。映射关系数量是 HTTP Server Port * 机器人数量。所以这种模型不可取。

2. NAT 打洞直接 P2P 通信。其实这个里面的内容很多，打洞的时候会牵涉到 4 种 NAT 类型，UDP/TCP 打洞方式，还有对应的两个协议 STUN/TURN。在 [Peer-to-Peer Communication Across Network Address Translators](http://www.bford.info/pub/net/p2pnat/index.html) 这篇论文里面有详细的介绍。在这里不细讲，但是这种方法对应当前的模型仍然不可用。因为对于 Symmetric NAT，无法保证 Client 的每次 Request 都统一相同的 Port。

3. 使用 Socket.io 作为长链接转发。当前项目就是用的这种方法，不过使用得有点丑陋。实际我们用了 JSON RPC 的方法调用。首先在 Socket.io Server 上定义转发 JSON 数据的事件，然后在机器人上面也定义事件来接受转发，然后用这些 HTTP Meta JSON RPC 访问本地服务器的 HTTP Server，最后再通过一个定义好的 JSON RPC 返回到 Client。这个过程维护了两条 Socket.io 的长连接，定义了很多转发事件，基本上就是一个基于 Socket.io 的 JSON RPC。说得有点乱，其实很简单，就像这样：

<iframe src="https://link.excalidraw.com/readonly/jQm4qvwzUbqzaP7uJC5C?darkMode=true" width="100%" height="500px" style="border: none;"></iframe>

不过最近看了一个做法 http://lifeofzjs.com/blog/2014/11/17/visit-server-behind-nat/。这个模型比 JSON RPC 更加简单和舒心。直接用一台服务器的 HTTP Server 进行转发 Socket.io 的 JSON RPC 到机器人，机器人返回的 Response 直接作为这台服务器的 HTTP Server Response 返回给 Client。这样只维护了一条 Socket.io 长连接，而且节约了 JSON RPC 的定义过程，而且通过公网的路由，`/robots/:id` 就可以对应访问不同机器人。简直舒心。

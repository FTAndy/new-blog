---
title: 'Sharedb 源码分析(待续)'
date: 2016/05/15
tag: javascript
author: Andy
---
## 入坑原因和暂时弃坑原因

之前帮一个公司做一个关于文档实时共享和编辑的项目，项目性质类似于 quip 和 google doc，核心功能使用到一个叫 sharejs 的库。

由于这个项目实时功能很强大，但是文档少而且很旧，所以想要深入了解一下内部原理，需要分析一下源码。此为背景，也为入坑原因。

但是！在读源码的时候，了解了一下这个项目的背景，发现坑有点深。

这个项目原本是被集成到一个叫 DerbyJS 的框架中（类似于 meteorjs 的实时通讯框架）和作为一个叫 Lever 硅谷公司的核心库，但是作者之前一段时间离开了这家公司，和由于一些私人原因放弃管理了这个项目很久，issue 一大堆，邮件列表不回复，所以项目一团糟。现在 sharejs 已经不维护，但是分出一个 sharedb 的项目，让 Lever 的 CTO 来管理，作者也在干自己的事没有管，那个 CTO 也是忙着管理公司，所以结果就是开发进度极慢，文档几乎没有，极少人关注这个库。

因为自己要忙找工作和毕业的事，所以把写到一半的笔记放下来，等以后有这个时间再继续研读源码，把这个坑填起来。

<!--more-->

以下为源码阅读笔记

### Client 部分

使用 websocket 或者 browserchannel(long polling) 实时通讯的方法，在 client 初始化，与服务器形成通信渠道，并让 sharedb 对象注册该通讯方法。

websocket 和 browserchannel 用了 w3c 规定的 websocket 回调函数接口：onopen, onerror, onclose, onmessage。当建立起通讯的时候回调 onopen，有信息来的时候回调 onmessage，错误信息回调 onclose，关闭通讯回调 onclose。

connection 对象 mixing 了 event emitter，可以自定义 emit 和相对应的 once。

初始化一个 connection 对象会绑定对应的 socket 对象（websocket 或者 browserchannel），并且定义好 client 端对 socket 的四个回调函数：onopen, onerror, onclose, onmessage。

在通讯的过程中，有四个状态：

- 'connecting': The connection is still being established, or we are still waiting on the server to send us the initialization message
- 'connected': The connection is open and we have connected to a server and recieved the initialization message
- 'disconnected': Connection is closed, but it will reconnect automatically
- 'closed': The connection was closed by the client, and will not reconnect
- 'stopped': The connection was closed by the server, and will not reconnect

doc 部分，分为 collection 和 docs，一个 collection 由多个 doc 组成，一个 doc 有自己的初创 id。

当 doc 需要 subscribe 到 server 的时候，会发送 subscribe 的动作和注册的 doc 名称。假如该文档没有

### Server 部分

server 可以绑定 mongo 或者 redis，或者不绑定，直接使用 memory，作为 doc 的数据暂存方法。

每当一个 client 初始化一个 socket，server 也会初始化一个对应的 client/session，这个 client/session 会对应上sharedb 的 agent。

server 会在每次会话定义一个 stream。sharedb 会将这个 stream 注册到 backend 对象中，并对应初始化一个 agent。初始化时，会向 client 发送确认初始化的信息，触发 client 的 onopen 事件。当有一端的 client 有动作并发送信息时，server 会以 stream 的形式读入该 client 的动作，并保存动作和版本号在 doc 对象当中，并向其他 subscribe 过的 agent 通过 socket 发送改动作信息。

### 库

hat: 随机数生成，随机一个 client/session id https://github.com/substack/node-hat

asyn: 异步函数集合 https://github.com/caolan/async

stream: 处理数据输入输出流 https://nodejs.org/api/stream.html https://github.com/jabez128/stream-handbook

ps: readable 读出时，用 on('data')事件监听数据，用 _read 函数对读出的数据进行数据格式化。writeable 被写入时，可以定义 _write 函数定义当有数据读入时，该如何处理数据。所以 sharedb 监听一个 stream ，当 backend 有操作的时候，会向该 stream 写入信息，然后该 stream 再触发 ws/browserchannel 的接口向客户端发送信息。

browserchannel: 基于长轮询的持久通讯 https://github.com/josephg/node-browserchannel

ws: 与 browserchannel 基本上有相同的开发 api，可以被 sharedb 使用 https://github.com/websockets/ws

arraydiff: 显示数组之间的差异 https://github.com/derbyjs/arraydiff

deep-is: 比较数据是否全等 https://github.com/thlorenz/deep-is

### 源文件

backend.js: 整个初始化的 backend，自服务器监听以后一直存在于 server，每当有一个新的 client 建立，都会对应建立一个新的 stream 监听到 backend，并创建一个新的 agent，并发送确认信息到 client，

agent.js: 解析 stream 中的写入信息，在 agent 触发相应的函数，使用 value 来返回。agent 变量在 server 端对应于 client 的一个 subscirbe。agent 会使用 _handleMessage 函数接收回来的动作并执行对应的私有函数

types.js: 注册默认ot类型，记录在 map 里。

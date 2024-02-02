---
title: '操蛋的浏览器缓存'
date: 2017/05/09
tag: browser
author: Andy
---
实际上我并不想写这么一篇关于浏览器缓存的文章，但是浏览器缓存太烦了，实际中经常会遇到，面试也经常问的，所以把关键的几个点记录一下。

缓存其实主要是服务器的锅，但是浏览器也可以做一点微小的工作。

<!--more-->

### 强缓存（服务器的锅）
假如想要缓存，也就是对于同一个资源文件 index.html，在浏览器下一次访问的时候不要网络请求，直接从浏览器缓存读取。那么首次访问该 index.html 文件时，服务器有两个 response 头可以设置：Cache-Control，Expires。由于 Cache-Control 会覆盖 Expires，而且 Expires 会有服务器和浏览器时间不统一的问题，一般都会直接看 Cache-Control。而且一般 Cache-Control 都只会设置 max-age，其他乱七八糟的 public，private 等等，遇到的时候再看文档。max-age 就是从返回时间开始，浏览器可以对该资源缓存的时间。所以，以后浏览器在这段时间访问该资源，就会直接从浏览器缓存获取，不会发一个 request，这段时间不会有协商缓存的事。但是，过了这段时间之后，就要从新发送 request，然后才有了协商缓存。

### 协商缓存（这也是服务器的锅）
强缓存时间过了之后，会向服务器发一个 request，然后再根据服务器的 response 来判断该资源是否需要获取新的，还是说按照老方法直接从本地读取。

协商缓存牵涉的 request 和 response 头有点多，下面分开讲。

#### Etag（这还是服务器的锅）
服务器第一次返回的资源的时候，很有可能把这个 Etag 带上，然后服务器就记下这个资源的 Etag。等到强缓存失效的时候，浏览器会向服务器发送 request 会带上 If-None-Match：Etag。然后服务器会根据这个 Etag 对比现在这个资源文件的 Etag，看是否相同，同则返回 304，然后浏览器拿到 304 状态码会继续读取本地缓存。不同则返回最新的文件。

Etag 也是覆盖 Last-Modified，所以还是看 Etag。

#### Last-Modified（这还是浏览器的锅啊，前端不背）
这也是服务器第一次返回的时候带上的，然后浏览器记下，等强缓存失效之后用 If-Modified-Since: Last-Modified 把这个带上，服务器在对比现在资源的 Last-Modified，然后判断是否相同，同则返回 304，浏览器再读取本地缓存。不同则返回最新的文件。

### 那前端可以干什么？
假如服务器没有把 response header 设置好，首先你可以直接去找后端骂他一顿，然后教他这么返回 response header。


假如他不屌你，前端也可以干点事情。协商缓存是干不掉了，要干必须设置 response header。假如不想要强缓存，可以设置 meta 标签：
```html
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate"/>
```
或者获取资源的时候加上时间戳，什么 index.html?t=时间戳。但是一般 index.html 文件都是固定了的，不可能加时间戳，可以写一个 redirect.html 的页面，专门加时间戳跳转。不过加这个 redirect.html 页面的时候记得加 meta，不然 redirect.html 也被缓存了。

### 总结
缓存也没想象那么复杂，要根据业务场景对应吧。

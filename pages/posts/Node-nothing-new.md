---
title: 'Node 根本就不是什么新东西'
date: 2017/01/17
tag: javascript
author: Andy
---
逛 github 看到 uWebSockets 作者 [Alex Hultman](https://github.com/alexhultman) 写的有关 Node 的话，很有感触。

第一篇 [The-real-history-of-async-networking](https://github.com/alexhultman/The-real-history-of-async-networking)，讲的是异步编程的历史，然后总结出 Node 并不是新概念，只是一群没有经验程序员的玩具。

> From the perspective of a PHP programmer it might be something new, but from the perspective of an experienced C programmer this piece of software is nothing more than a simplification of common (now ancient) async techniques wrapped up in JavaScript for non-experienced programmers to toy with

确实也无可厚非。

第二篇 [Don-t-bother-counting-those-stars](https://github.com/alexhultman/Don-t-bother-counting-those-stars)，讲的是低质量的库，假如做了宣传，star 飙升，但是并没有否定这是一个低质量库的现实。而很多高质量的库因为没有宣传，star 少，但是实用。总结为不要看 star 数量而用一个库。

> You won't get many stars with marketing like “We are not the fastest but we are pretty efficient, no wait, we are actually not that fast”. It has to be something like “FEATURING THE FASTEST AND MOST RELIABLE REAL-TIME ENGINE”. Yeah, that one will sell. In fact, this is the marketing of Socket.IO, a project with 28k stars.

确实，最近用到 Socket.io，质量确实不好，而且很久没人维护。

Node 09 年刚出现的时候被大吹特吹，被贴上异步事件驱动，高并发等标签，入门门槛极低，大量前端都可以跳到 Node写一下。一些无高质量的基础库涌现，库作者会吹 B，而且前端的人很多，这些库 star 数飙升，例如 Socket.io。但是到现在出现了性能低，无人维护等现象。

Node 要成为下一个 Ruby 了？

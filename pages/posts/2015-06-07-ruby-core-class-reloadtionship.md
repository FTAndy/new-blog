---
title: 'Ruby core class reloadtionship'
date: 2015/06/07
tag: ruby
author: Andy
---

### Ruby 核心类的关系
Ruby 主要由标准库和核心类构成，有一些标准库会在你打开 irb 的时候自动加载，有些则是要自己手动加载，不过所有核心类都是默认配置的。  

<!--more-->

上次在stackoverflow 找到这幅图之后一直想要把这幅图清晰化，这样就可以清晰知道 Ruby 核心类的骨架是怎么样的了，用的时候也可以在这个骨架上 中找到自己想要的类。

![](https://ruby-china-files.b0.upaiyun.com/photo/2015/39957ccd0f08204244a385a4134ff87c.jpg)  

### 自己画图
首先Ruby2.2.0核心类总共有123个类，先筛选出比较重要的类  

- ConditionVariable 线程同步互斥资源化
- Queue 线程之间用队列通讯
- SizedQueue 限制了Queue的大小，当超出Queue大小时发生阻塞
- Array 基本的数组
- Bignum 比Fixnum大的数的类型
- BasicObject Object的父类
- Object 所有对象最终都继承自Object
- Module 模块
- Class 类

### TODO

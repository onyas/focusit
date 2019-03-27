---
title: 一文读懂Back Pressure 
date: 2018-03-27T19:12:20+08:00
categories:
  - Architecture
tags: 
  - java
  - back-pressure	 
---

最近经常看到back-pressure这个词，所些想要深入了解一下，以下是学习的一点心得。

## 什么是back-pressure

back-pressure这个词是来源于工程概念，当气流或液体在管道中运输时，由于管道变细或者受到其他阻碍，导致出现了下游向上游的逆向压力，这种情况就称为back pressure，也称作向后的压力。

在计算机行业，back pressure 通常用来描述当数据在传输中由于下层的buffer满了，导致上层服务无法继续接收数据的现象。


## 为什么会产生back-pressure

我们在开发中遇到的一个很现实的问题是所有的资源都有瓶颈，比如CPU，内存，磁盘等。假如我们部署的一个程序能够同时支持1W人在线，这里的1W就是我们系统的buffer，那么当有10001人同时访问的时候，就超过我们程序的最大承受能力，这就是back pressure现象。

这种现象在现实中很容易出现，尤其现在遍地都是微服务，上游服务大量请求下游服务，而下游服务因为瓶颈处理不过来的时候就出现了。

## 如何处理back-pressure

当back pressure出现的时候，合理的处理方式是下游系统通知上游系统，说我现在处理不了，不要请求太多过太快。

但是实际情况是大多不可控，比如电商系统中的秒杀场景，我们阻止不了用户发起请求，但是我们可以丢弃用户的请求。因为如果不丢弃的话会意味着系统崩溃。

## 其他场景下的back-pressure

TCP流量控制中就用到了back pressure，当数据在两个节点之前通过tcp传输时，如何确定每次数据包传输的大小呢？这里就是接收方把自已的buffer反馈给发送方，这样发送方就可以跟据接收到的buffer大小来确定每次发送包的大小，确保不会给接收方太大的压力。

## 引用
https://en.wikipedia.org/wiki/Back_pressure
https://www.zhihu.com/question/49618581
https://www.cnblogs.com/itrena/p/9023589.html
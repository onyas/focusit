---
title: How about introducing Chao Engineering
date: 2020-02-25T19:12:20+08:00
categories:
  - architecture
tags: 
  - chao-engineering
---


## Context and Problem

Baiscally the system behaves normally when the amount of data is small, but the system becomes unstable as the amount of data increases, at the same time, we have more and more components. Some issues are not visible with usual tesing, although we have big data testing,but it’s not enough.

## Solution

By intorducing Chao Engineering, we could involve some faults manully, for example, like network latency, limited memory, full disk, damaged databas,etc. In this way we could find the weakness of our system. What we missed about monitor and what we could resolve beforehand.

## Issues and considerations

- Usual testing is still needed, chao Engineering is complement traditional testing.
-  

## Refrence

[chaos-engineering-explained](https://blog.newrelic.com/engineering/chaos-engineering-explained/)

[chaos-engineering-the-history-principles-and-practice/](https://www.gremlin.com/community/tutorials/chaos-engineering-the-history-principles-and-practice/)

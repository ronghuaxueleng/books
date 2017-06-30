---
title: Promise对象 
tags: Promise,异步,回调,ES6
grammar_cjkRuby: true
---


## Promise 的含义
>`Promise`是异步编程的一种解决方案，比传统解决方案（事件和回调函数）更合理和更强大

所谓`Promise`，简单说就是一个容器，**里面存放着某个未来才会结束的事件（通常是一个异步操作）的结果**。
从语法上说，**`Promise`是一个对象，从它可以获得异步操作的消息**。`Promise`提供统一的`API`，各种异步操作都可以用同样的方法进行处理。

`Promise`有以下两个特点:
1、**对象的状态不受外界影响**。`Promise`对象代表一个异步操作，有三种状态，`Pending`（进行中）、`Resolved`（已完成）、`Rejected`（已失败）
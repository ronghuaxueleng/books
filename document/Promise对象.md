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
1、**对象的状态不受外界影响**。`Promise`对象代表一个异步操作，有三种状态，`Pending`（进行中）、`Resolved`（已完成）、`Rejected`（已失败），只有异步操作结果才可以决定当前是哪一种状态，任何其他操作都无法改变其中的状态。

2、**一旦状态改变，就不会再改变，任何时候都可以得到这个结果**。`Promise`对象的状态改变只有两种可能：从`Pending`变成`Resolved`和从`Pending`变成`Rejected`。

>有了`Promise`对象，就可以**将异步操作以同步操作的流程表达出来**，避免了层层嵌套的回调函数。此外`Promise`提供了统一的接口，是的异步操作更加容易。

`Promise`也有一些缺点：
1、**无法取消`Promise`，一旦创建他就会立即执行，无法中途取消。**
2、**如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部**
3、**当处于`Pending`状态时，无法得知目前进展到哪一阶段（刚刚开始还是即将完成）**


## 基本用法
> ES6规定，`Promise`对象是一个构造函数，用来生成`Promise`实例，
下面代码创造了一个`Promise`实例：
```javascript
var promise = new Promise(function(resolve, reject) {
  // ... 一些操作

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

*`Promise`构造函数接受一个函数作为参数，该函数的两个参数是两个函数，分别是`resolve`和`reject`。
`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从`Pending`变为`Resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从`Pending`变为`Rejected`）*

`Promise`实例生成以后，可以用`then`方法分别指定`Resolved`状态和`Reject`状态的回调函数。

```javascript
promise.then(function(value) {
  // 成功
}, function(error) {
  // 失败
});
```
*`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`Resolved`时调用，第二个回调函数是`Promise`对象的状态变为`Rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。*

下面是一个用Promise对象实现的 Ajax 操作的例子。
```javascript
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
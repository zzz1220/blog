---
title: 发布订阅模式的javascript实现
tags:
  - JavaScript
abbrlink: 4ec38d83
date: 2019-05-27 11:20:25
---

### 发布订阅模式

在[软件架构](https://zh.wikipedia.org/wiki/软件架构)中，**发布-订阅**是一种[消息](https://zh.wikipedia.org/wiki/消息)[范式](https://zh.wikipedia.org/wiki/范式)，消息的发送者（称为发布者）不会将消息直接发送给特定的接收者（称为订阅者）。而是将发布的消息分为不同的类别，无需了解哪些订阅者（如果有的话）可能存在。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者（如果有的话）存在。

发布/订阅是[消息队列](https://zh.wikipedia.org/wiki/消息队列)范式的兄弟，通常是更大的[面向消息中间件](https://zh.wikipedia.org/w/index.php?title=面向消息中间件&action=edit&redlink=1)系统的一部分。大多数消息系统在[API](https://zh.wikipedia.org/wiki/应用程序接口)中同时支持消息队列模型和发布/订阅模型，例如[Java消息服务](https://zh.wikipedia.org/wiki/Java消息服务)（JMS）。

这种模式提供了更大的网络[可扩展性](https://zh.wikipedia.org/wiki/可扩展性)和更动态的[网络拓扑](https://zh.wikipedia.org/wiki/网络拓扑)，同时也降低了对发布者和发布数据的结构修改的灵活性。



优点：

1. 松耦合：发布者与订阅者[松耦合](https://zh.wikipedia.org/wiki/松耦合)，甚至不需要知道它们的存在。
2. 可扩展性：通过并行操作，消息缓存，基于树或基于网络的路由等技术，发布/订阅提供了比传统的客户端–服务器更好的[可扩展性](https://zh.wikipedia.org/wiki/可扩放性)。



### `JavaScript`里的发布订阅模式

在`dom`事件里面，有一种事件监听的模式就是基于发布订阅设计的，比如非常常见的给节点添加点击事件

```javascript
var button = document.getElementById("button")
button.addEventListener('click',() => {alert('click')},false)
```

这样的写法好处很明显，分离事件触发和事件内容解耦。



在`node`里面，还有`EventEmitter`这个类，用来实现发布订阅模式，来处理不同对象之间的事件调用。

```javascript
const EventEmitter = require('events');	
const MyEventEmitter extends EventEmitter

const myEventEmitter = new MyEventEmitter
myEventEmitter.on('event',() => {
    console.log("target this event")
})
myEventEmitter.emit("event")
// target this event
```

我之前用`BackBone`构建单页面应用的时候，就广泛用到了发布订阅模式处理不同组件之间的数据交互，但是这种方法如果滥用，会造成事件交互逻辑的不清晰，所以，父子组件数据交互还是推荐用回调的方式来做。

### 怎么实现`eventEmitter`

下面是简单实现`eventEmitter`的`JavaScript`代码

```javascript
function EventEmitter() {
    this._event = {}
    this.maxListenerNum = 10
}

EventEmitter.prototype.on = function(type,callback) {
    if(typeof callback !== "function") {
        throw new TypeError('callback must be a function')
    }
    if(!this._event) {
        this._event = Object.create(null) // 这里判断不存在就生成一个，如果是继承的，不会继承这个 _event
    }

    if(this._event[type] ) {
        if(this._event[type.length > this.maxListenerNum] ) {
            throw new Error("too long")
        }
        this._event[type].push(callback)
    } else {
        this._event[type] = [callback]
    }
}


EventEmitter.prototype.once = function (type,callback) {
    let warp = (...args) => {
        callback(...args)
        this.removeListener(type,warp)
    }
    warp.flag = callback
    this.on(type,warp)
}

EventEmitter.prototype.removeListener = function(type,callback)  {
    if(this._event[type] ) {
        this._event[type].splice(this._event[type].indexOf(callback),1)
    }
}

EventEmitter.prototype.removeListeners = function(type){
    this._event[type] = null
}

EventEmitter.prototype.emit = function(type,...args) {
    if(this._event[type]) {
        this._event[type].forEach(fn => {
            fn(...args)
        });
    }

}
```

测试代码

```javascript
var my = new EventEmitter

my.on('a',(a) => {
    console.log(a);
})


my.once('once', (a) => {
    console.log(a);
})

let add = (a,b) => {
    console.log(a+b); 
}
my.on('b',add)
my.emit('a',1)
my.emit('a',2)
my.emit('b',1,2)
my.emit('once',1)
my.emit('once',2)
my.removeListeners('a')
my.emit('a',1)
my.removeListener('b')
my.emit('b',1,2)

/* 输出
1
2
3
1
*/
```



参考：

1.[https://zh.wikipedia.org/zh-hans/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85](https://zh.wikipedia.org/zh-hans/发布/订阅)

2.<https://segmentfault.com/a/1190000015762318>


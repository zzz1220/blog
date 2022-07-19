---
title: javaScript异步操作进化曲
tags:
  - javaScript
  - promise
abbrlink: 2b77e8c0
date: 2018-07-11 21:54:30
---
> 老生常谈，今天也来总结下`javaScript` 中异步操作的历史。

*TL;DR*  
1.同步和异步区别  
2.经典的回调函数    
3.简单明了的`promise`函数  
4.未来的主流`async`函数  

## 同步和异步的区别
异步简单来说就是做一件事时，做到一半可能需要等待系统或服务处理之后才会得到响应和结果，此时可以转去做另一件事，等到获得响应之后在去执行剩下一半的事情。反之同步就是一直等到响应然后接着做事，中间不会跳去做别的事。

## 回调函数
就是我们经常用到的回调函数，最开始用ajax 的时候，会写出下面样子的代码。

```javascript
$.ajax({
    url:"path",
    success:function() {
        // do something
    }
})

```

缺点是读起来跳跃性太强，不好跟踪代码。而且在前后依赖的时候，形成回调地狱，一旦修改一个，嵌套的都要改。另外就是无法合并多个异步代码。

## 简单明了的`promise`函数 
Promise的引入就解决了以上这些问题,首先来看下Promise的简单用法：

```javascript
let p = new Promise(function(resolve, reject) {
    resolve(100)；
})；

p.then(function(data) {
    console.log('data', data);
}, function(err) {
    console.log('err', err);
});

```
可以看到Promis通过then的链式调用解决了嵌套回调的问题，在用法上Promise的构造函数会接受一个executor函数，这个函数带有两个参数resolve和reject，两个参数背后其实就是两个函数，而通过Promise构造函数创建出来的对象会保存一个status属性，resolve会做的事就是将这个属性从初始化的pending转为resolved，而reject则是转为rejected，同时两个函数都可以接受一个参数，作为之后then中回调函数的参数传入，那么在then方法中我们可以看到它接收两个参数，第一个就是成功resolved之后会调用的回调函数，第二个就是rejected的回调函数。
这里注意的是，只要状态转为resolved或rejected之中的其中一个，那么当前promise对象就不能再转变状态了。之后不管调resolve还是reject都会被忽略。
另外，上面所说Promise是可以支持链式调用的，所以then是可以多次调用的，但是因为刚刚所说状态不可转变的问题，所以链式调用每次then返回的不是当前的Promise对象而是一个新的Promise对象，那么第2次then的状态又是怎么决定的呢，第一次then中无论是成功的回调还是失败的回调只要返回了结果就会走下一个then中的成功，如果有错误走下一个then的失败。

此外，`promise` 还有：
```javascript
Promise.all()
Promise.race()
Promise.resolve()
Promise.reject()
```

这些api
[Promise 对象](http://es6.ruanyifeng.com/#docs/promise)。

关于promise的实现，可以看下PromiseA+的原则。下面是一个实现：

<!-- Get UTF-8 Size (ANSI C) -->
<!-- Begin -->
<script src="https://gist.github.com/x1nes/5fad2985f43f235ed59702b805a7d824.js"></script>
<!-- End -->



## async+await
async+await就是目前为至，异步的最佳解决方案，它同时解决了
其实，它是`Generator`函数的语法糖，之后我会单独写一篇博文介绍`Generator`函数和它的异步用法。
回调地狱
并发执行异步，在同一时刻同步返回结果 Promise.all
返回值的问题
可以实现代码的try/catch;

```javascript
let bluebird = require('bluebird');
let fs = require('fs');
let read = bluebird.promisify(fs.readFile);

// 用async来修饰函数，aysnc需要配await, await只能接promise
// async和await(语法糖) === co + generator
async function r() {
    try{
        let content1 = await read('./2.promise/100.txt', 'utf8');
        let content2 = await read(content1, 'utf8');
        return content2;
    } catch(e) { // 如果出错会catch
        console.log('err', e)
    }
}

// async函数返回的是promise
r().then(function(data) {
    console.log('flag', data);
}, function(err) {
    console.log(err);`
})

```


此外，还有Generator 实现的，利用`co`库实现 next方法嵌套取值执行，本人很少用到，就不详细介绍了。 
可以看[Generator 函数的异步应用](http://es6.ruanyifeng.com/#docs/generator-async)
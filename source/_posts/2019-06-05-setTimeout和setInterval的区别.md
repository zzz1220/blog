---
title: setTimeout和setInterval的区别
tags: javaScript
abbrlink: 64e435bc
date: 2019-06-05 18:55:13
---
> 我们再前端开发中需要实现定时任务或者`js`动画，都要用到`setTimeout`和`setInterval`z这两个函数，但是对于他们的原理，如果不清楚，往往会造成一些意想不到的问题。

### `setTimeout`

`setTimeout`方法设置一个定时器，该定时器在定时器到期后执行一个函数或指定的一段代码。

 语法

```javascript
var id = setTimeout(function[,delay,param1...])
var id = setTimeout(function[,delay])
var id = setTimeout(code[,delay])
```

#### 参数

`function`

[`function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/function) 是你想要在到期时间(`delay`毫秒)之后执行的[函数](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Function)。

`code`

这是一个可选语法，你可以使用字符串而不是[`function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/function) ，在`delay`毫秒之后编译和执行字符串 (使用该语法是**不推荐的,** 原因和使用 [`eval()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval)一样，有安全风险)。

`delay `可选

延迟的毫秒数 (一秒等于1000毫秒)，函数的调用会在该延迟之后发生。如果省略该参数，delay取默认值0，意味着“马上”执行，或者尽快执行。不管是哪种情况，实际的延迟时间可能会比期待的(delay毫秒数) 值长，原因请查看[Reasons for delays longer than specified](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout#Reasons_for_delays_longer_than_specified)。

`param1, ..., paramN` 可选

附加参数，一旦定时器到期，它们会作为参数传递给[`function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/function) 

#### 返回值

返回值`timeoutID`是一个正整数，表示定时器的编号。这个值可以传递给[`clearTimeout()`](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/clearTimeout)来取消该定时器。

#### 用途

举个例子，三秒后跳转新页面

```javascript
setTimeout(function() {
    location.href = "url"
},3000)
```
在函数执行前取消
```javascript
var key;
function myFunction() {
    key = setTimeout(function(){ console.log("fn") }, 3000);
}
function myStopFunction() {
    clearTimeout(key);
}
```

如果你只是掌握了上面的用法，不了解它的原理，就会犯一些错误

比如一个常见的面试题



```javas
for (var i = 1;i <= 5;i ++) {
  setTimeout(function timer() {
      console.log(i)
  },i * 1000)
}
```

问打印出来的是什么？

如果对闭包和作用域不了解的同学会以为是每隔一秒就输出1,2,3,4,5。但实际上是每隔一秒输出了6。

原因就是作用域，在`javascript`里面没有块作用域，因此，当`setTimeout`里面的函数执行的时候，所有的i都是`for`循环里面的`i`，此时，`i`是6。

另外，说到`setTimeout`就一定要说一下事件循环，这个概念比较复杂，下面简单说一下，

`setTimeout`运行后，会把第一个参数的函数在第二个参数设置的时间之后，将其放入异步队列里面，然后等到主线程函数栈为空，在执行这个函数。

这里有个需要注意的地方，

> HTML5 标准规定了`setTimeout()`的第二个参数的最小值，即最短间隔，不得低于4毫秒。如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。

这意味着，即使你把第二个参数设置为0，也要最少等个4ms才能进入异步队列。另外，就算进入异步队列,也要等队列里面其他未执行的函数执行完，因此，可不要以为只要设置一个时间，就一定可以准时执行这个函数。

那么，怎么让上面的代码打印出0,1,2,3,4,5呢？

可以用下面的方法。

1.`es6`的`let`关键词

`let`是`es6`的块作用域的声明关键词。

```javascript
for(let i = 0;i<5;i++) {
  setTimeout(function timer(){
    console.log(i);
  }, i * 1000);
}
```

2.利用闭包

```javascript
for(var i = 0;i<5;i ++) {
  (function(i){
    setTimeout(function timer() {
      console.log(i)
    }, i * 1000);
  })(i);
}
```

3.利用第三个参数

```javascript
for (var i=1; i<=5; i++) {
  setTimeout( function timer(i) {
    console.log(i);    
   }, i*1000,i );
}
```

tips:

`setTimeout`第一个参数函数里面的`this`是`window`。

###　setInterval

**setInterval()** 方法重复调用一个函数或执行一个代码段，在每次调用之间具有固定的时间延迟。

语法

```javascript
let intervalID = window.setInterval(func, delay[, param1, param2, ...]);
let intervalID = window.setInterval(code, delay);
```

参数

- `intervalID` 是此重复操作的唯一辨识符，可以作为参数传给`clearInterval``()`。
- `func` 是你想要重复调用的函数。
- `code` 是另一种语法的应用，是指你想要重复执行的一段字符串构成的代码(使用该语法是**不推荐**的，不推荐的原因和[eval()](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/eval#Don't_use_eval!)一样)。
- `delay` 是每次延迟的毫秒数 (一秒等于1000毫秒)，函数的每次调用会在该延迟之后发生。和[setTimeout](https://developer.mozilla.org/en-US/docs/DOM/window.setTimeout#Minimum_delay_and_timeout_nesting)一样，实际的延迟时间可能会稍长一点。

返回值

返回一个 intervalID。可以用clearInterval函数来取消定时任务。

**这里需要注意，于setTimeout返回的id公用一个id池**

通常，这个函数用来执行周期执行的函数，比如动画之类的效果。

mdn里面有个打字机效果的例子。

[打字机效果](https://mdn.mozillademos.org/files/3997/typewriter.html)

和`setTimeout`一样，也是加入到异步队列里，等待合适的时机运行。

> 当使用`setInterval`时,仅当(在队列中)没有该定时器的任何其他代码实例时,才将定时器代码添加到队列中,引用JavaScript高级程序设计第三版书中语句

(即：当前一个定时器代码执行时,紧跟后面的第一个定时器代码将添加到队列中,等待执行,再后面的定时器代码不会添加到队列中)

所以，如果你的函数（第一个参数）执行时间比较长，但是你的执行间隔时间设置的比较短，可不要以为可以精准的在预料的时间点会执行设置的函数。

此外，第二个参数也有一个最短的默认参数，在`chrome`里面是7ms。

当定时器代码执行时间超过指定间隔,那么某些定时器代码就会被跳过(即后面的定时器代码不会被添加到队列中),前一个定时器代码执行完毕后,队列中的定时器代码立刻执行,各定时器之间的代码执行没有间隔。这时，需要使用链式`setTimeout`。

```javas
setTimeout(function(){
    //要执行的代码 
    setTimeout(arguments.callee,2000);                   
},2000);
```



上面的代码就是用`setTimeout`来模拟`setInterval`,如果面试官问你怎么用`setTimeout`来解决定时器问题，可以用上面的代码来回答哦。



如果要求在每隔一个固定的时间间隔后就精确地执行某动作，那么最好使用`setInterval`，

比如函数执行的时间比较短的，用来实现动画效果。

如果每次函数的调用需要繁重的计算以及很长的处理时间，而且不希望他们之间互相干扰，那么最好使用`setTimeout`。
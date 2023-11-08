---
title: javaScript柯里化
layout: post
tags: JavaScript
category: code
comments: true
abbrlink: 2b34be88
---
> 一直对函数式编程非常感兴趣，因此时常会留意下关于函数式编程的概念，之前一直听到柯里化，但是只闻其名，知道他可以接受一个函数，并且返回值也是函数，返回的函数可以继续接受参数运算，直到出发原始函数的执行条件，并没有认真研究，今天就花点时间好好研究下，看看如何写出符合柯里化（curry）的函数，

### 什么是柯里化？
在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。这个技术由 Christopher Strachey 以逻辑学家 Haskell Curry 命名的，尽管它是 Moses Schnfinkel 和 Gottlob Frege 发明的。

>柯里化允许我们把函数与传递给它的参数结合，产生一个新的函数。[引自《JavaScript语言精髓》，第43页 柯里化]

通俗一点，就是之前的函数是这样的
```javascript
function add(x,y) {
	return x+y;
}
```
而柯里化后
```javascript
var curryAdd = curry(add)
```

要计算的时候，就是
```javascript
cuurryAdd(1)(2)
```

看上去很不错吧，在函数式编程里面，简化了代码的复杂性，代码量变小，看起来更加优雅，可以达到延迟计算的方法，不断传入参数，最后执行；另外，根据网上的说法:
>函数柯里化允许和鼓励你分隔复杂功能变成更小更容易分析的部分。这些小的逻辑单元显然是更容易理解和测试的，然后你的应用就会变成干净而整洁的组合，由一些小单元组成的组合。

关于这一点，我还在自己品味。是不是可以省略多参函数的参数，带入具体的函数以提供更细分的方法调用？

### 通用的柯里化方法：
[代码引自《JavaScript: Novice to Ninja》，第319页 A General Curry Function]   
```javascript
function curry(func) {
  var fixedArgs = [].slice.call(arguments,1);
  return function() {
    args = fixedArgs.concat([].slice.call(arguments))
    return func.apply(null, args);
  };
}
```   

运行方法：
```javascript
function divider(x,y) {
  return x/y;
}
reciprocal = curry(divider,1);
reciprocal(2);
```

执行reciprocal(2)相当于执行curry(divider,1)(2)。然后看curry方法通过[].slice方法去掉了传入的第一个参数，即function divider(x,y) {return x/y;}，之后保留了变量x=1并返回一个新的函数，这是一个匿名函数，同时传入变量y=2，这时fixedArgs.concat([].slice.call(arguments))将之前传入的变量x和现在传入的变量y重新组合成一个数组，最后执行函数divider，这时使用的参数是组合后的[1,2]。整个curry方法主要工作就是组织两次传入的参数，然后通过apply方式调用函数。

最后，如果利用es6的新特性的话，就可以一行写出一个柯里化函数
```javascript
var currySingle = fn => ((limit) => judgeCurry = (...args) => args.length >= limit ? fn.apply(null, args) : (...args2) => judgeCurry.apply(null, args.concat(args2)))(fn.length)
```

摘自[一行写出javascript函数式编程中的curry] https://segmentfault.com/a/1190000008248646





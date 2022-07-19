---
title: javascript编码技巧
layout: post
tags: javaScript
category: code
comments: true
abbrlink: 6cf112d6
---
> 在平时的工作中，一些前端的工作，经常要用的到javaScript，而且现在自己正在学习react.js和node.js,所以现在把自己平时遇到的写代码猜到的一些坑记录下来，此篇文章会随时更新。

------------

# 1. 避免TypeError异常
有时从后台取到的json对象，里面某个属性可能不存在，就会返回undefined,比如
```javascript
    var cat = {name: "mm"}
    cat.color //undefinedd
    cat.color.code // 报异常TypeError （去拿颜色代码）
    //可以用下面的代码
    var code = cat.color && cat.color.code // && 具有短路功能,前面是false就不去执行后面的,前面是true，得到后面的值
```

# 2. 高阶函数的使用
把函数作为参数传入，这样的函数称为高阶函数.
高阶函数英文叫Higher-order function。 对于结集合的操作，遍历做一些操作，用高阶函数很简洁也可以写成链式语法。

`map`

```javascript
var l = [1,2,3,4,5,6,7]
l = l.map(v => v*v)

[1, 4, 9, 16, 25, 36, 49]
```

   **map就是对集合里每一个元素进行操作,比如求每一个元素的平方。**

`filter`
```javascript
var l = [1,2,3,4,5,6];
l = l.filter(v => v%2 === 0);

[2, 4, 6]
```
**filter过滤出集合重符合某些条件的元素，得到这些元素组成的新的集合。** 

`every`
```javascript
var l = [1,2,3,4,5,6,7,8]
l.every(v => v<3)

false
```
**every 对每个元素执行操作，全部为true，就返回true，有一个false就返回false,它只对数组中的非空元素执行指定的函数，没有赋值或者已经删除的元素将被忽略.用来判断集合是不是每一个都符合筛选条件。**

`some` 
**类似上面的every，它只要有一个符合就返回true,用来判断集合内有没有符合条件的元素。**

`reduce`
```javascript
var numbers = [65, 44, 12, 4];
 
function getSum(total, num) {
    return total + num;
}
function myFunction(item) {
    document.getElementById("demo").innerHTML = numbers.reduce(getSum);
}
```
这个函数：
`array.reduce(function(total, currentValue, currentIndex, arr), initialValue)`

|  参数         | 描述                          |
| ------------ | ----------------------------- |
| total  |必需。初始值, 或者计算结束后的返回值。   |
|  currentValue | 必需。当前元素                 |
| currentIndex  |  可选。当前元素的索引          |
| arr  |可选。当前元素所属的数组对象。            |

**reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终为一个值，是ES5中新增的又一个数组逐项处理方法**


注意 sort函数：
`arr.sort()` 默认情况下是用unicode 码来排序，也可以传入一个返回值为"数值"的函数:
```javascript
function compare(a,b){
    return a-b;
}
```
因此，如果不能直接减的变量，要先判断大小，在返回（-1，1,0）等数值。

# 3.特殊的技巧
js是一个很神奇的编程语言，有各种骚操作，下面举几个例子：

向下取整
`var a = ~~1.3;//1`

转数字
`var a = +"22";//22`

日期转数字
`var date = +new Date()`

设置默认值
`var b = "cat"||"default"`;

类数组对象转成数组
`var arr = Array.prototype.slice.call(arguments)`

漂亮的随机码

    Math.random().toString(16).substring(2); //14位
    Math.random().toString(36).substring(2); //11位
交换值
`a= [b, b=a][0];`

匿名函数立即执行，可以避免全局变量的污染

    (function(){
     //code here
    })();

怎么在手机上调试js代码?  
这里推荐以下 微信 团队做的 js调试利器--vconsole.  
`npm install vconsole`  
或者  
```javascript
<script src="path/to/vconsole.min.js"></script>
<script>
  // init vConsole
  var vConsole = new VConsole();
  console.log('Hello world');
</script> 
```

## 4.编程习惯
`if else`
不要小看`if else` 语句，尤其是在有异常处理的时候。
有些同学喜欢这样写
```javascript
if(/*条件*/){
    //正常逻辑
}else{
    //出错的时候处理
}
```
这样很容易忽略错误的条件,改成这样
```javascript
if(/*出错的条件*/){
    // 对错误处理
}

//正常逻辑
```

这样就很清晰了。突出错误的条件，就会避免这些不合法的输入。

变量的初始化
`var a = "";`
`var b = [];`
`var c= {};`
直接用字面表达展示数据类型,避免用new 操作符。

简单的技巧提高网页函数复用
我曾看到过长达两百多行的javascript函数，只是为了处理页面上的一个点击事件，其实有个很简单的范式，一个函数用来处理事件，接受事件，另外一个函数用来处理业务，参数来自上个函数。这样一般情况下就会提高函数复用。

声明局部变量
每一个js函数都会带有一个叫做[[Scope]]的内部属性，也就是该函数的作用域链，它决定了哪些数据能被函数访问。
一个函数若要使用一个变量，它会从最近的地方，也就是定义在函数内部的局部变量里面去找；若没有找到，则往更远处的全局变量（或者上一级作用域）里面去找。恰恰是这个“找”的过程，产生了性能的问题。 因此，可以把一个较深的变量赋值给一个局部变量，在函数内部直接调用这个局部变量来提升性能。

使用jquery选择器的时候,把选到的jquery对象缓存到一个变量里,避免每次使用到这个元素都重新查询。
如：
```javascript
var $input = $("input");
```

 ## 










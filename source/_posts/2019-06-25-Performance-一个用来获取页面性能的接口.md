---
title: Performance--一个用来获取页面性能的接口
date: 2019-06-25 14:29:02
abbrlink: 94e1353c
tags:
	 - JavaScript
---

> 最近被问到一个问题，如何判断页面加载时间，我第一时间想到的是打开`devtool`看下`Network`下面
>
> `DOMContentLoaded`和`load`分别用了多久，如果自己写的话，要用到`window.onload`函数，但是实际上，web已经给出了`api`，获取页面性能，并且大部分现代浏览器也支持这个`api`.

### **Performance**简介

根据`mdn`里面的说法:

> **Performance** 接口可以获取到当前页面中与性能相关的信息。它是 High Resolution Time API 的一部分，同时也融合了 Performance Timeline API、[Navigation Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_timing_API)、 [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) 和 [Resource Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API)。

并且，这些数据是只读的。



`preformance`对象有三个标准属性，和一个`chrome`添加的非标准库--[`performance.memory`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/memory)，用来获取到基本内存的使用情况，一般来说不要用这个非标准的`api`。

:thumbsdown:[`Performance.navigation`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/navigation) [只读]

[`PerformanceNavigation`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceNavigation) 对象提供了在指定的时间段里发生的操作相关信息，包括页面是加载还是刷新、发生了多少次重定向等等。Not available in workers.

:thumbsdown:[`Performance.timing`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/timing) [只读]

[`PerformanceTiming`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming) 对象包含延迟相关的性能信息。Not available in workers.

[`performance.memory`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/memory) 

其是 Chrome 添加的一个非标准扩展，这个属性提供了一个可以获取到基本内存使用情况的对象。**不应该**使用这个非标准的 API。

[`Performance.timeOrigin`](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance/timeOrigin) [只读] 

返回性能测量开始时的时间的高精度时间戳。



关于更精确的资料请查看`mdn`官方文档。

[Performance](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance)	

### 如何根据`Performance.timing`获取加载时间



在此之前，读者需要明白一点浏览器工作的基本流程和原理，比如经常在面试时被问到的“从url输入到页面加载经过了什么流程？”

这里有几篇非常详细的文章（最起码是我遇到的最详细的，如果你看到过更详细的，或者还有其他补充，请在下方留言，万分感谢:kissing_smiling_eyes: ），介绍了这个流程。

[从输入URL到页面加载发生了什么](https://segmentfault.com/a/1190000006879700)
[浏览器的工作原理：新式网络浏览器幕后揭秘](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)	

也就是说，你最起码要知道`dns`解析，`http`请求，页面渲染等认识。

`timing`就是从输入`url`到页面展示的全过程的时间统计，单位是毫秒，只读。



:heavy_plus_sign: 属性：

[`PerformanceTiming.navigationStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/navigationStart) 只读

是一个无符号long long 型的毫秒数，表征了从同一个浏览器上下文的上一个文档卸载(unload)结束时的UNIX时间戳。如果没有上一个文档，这个值会和PerformanceTiming.fetchStart相同。

[`PerformanceTiming.unloadEventStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/unloadEventStart) 只读

是一个无符号long long 型的毫秒数，表征了`unload`事件抛出时的UNIX时间戳。如果没有上一个文档，or if the previous document, or one of the needed redirects, is not of the same origin, 这个值会返回0.

[`PerformanceTiming.unloadEventEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/unloadEventEnd) 只读

是一个无符号long long 型的毫秒数，表征了`unload`事件处理完成时的UNIX时间戳。如果没有上一个文档，or if the previous document, or one of the needed redirects, is not of the same origin, 这个值会返回0.

[`PerformanceTiming.redirectStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/redirectStart) 只读

是一个无符号long long 型的毫秒数，表征了第一个HTTP重定向开始时的UNIX时间戳。如果没有重定向，或者重定向中的一个不同源，这个值会返回0.

[`PerformanceTiming.redirectEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/redirectEnd) 只读

是一个无符号long long 型的毫秒数，表征了最后一个HTTP重定向完成时（也就是说是HTTP响应的最后一个比特直接被收到的时间）的UNIX时间戳。如果没有重定向，或者重定向中的一个不同源，这个值会返回0.

[`PerformanceTiming.fetchStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/fetchStart) 只读

是一个无符号long long 型的毫秒数，表征了浏览器准备好使用HTTP请求来获取(fetch)文档的UNIX时间戳。这个时间点会在检查任何应用缓存之前。

[`PerformanceTiming.domainLookupStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domainLookupStart) 只读

是一个无符号long long 型的毫秒数，表征了域名查询开始的UNIX时间戳。如果使用了持续连接(persistent connection)，或者这个信息存储到了缓存或者本地资源上，这个值将和 `PerformanceTiming.fetchStart一致。`

[`PerformanceTiming.domainLookupEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domainLookupEnd) 只读

是一个无符号long long 型的毫秒数，表征了域名查询结束的UNIX时间戳。如果使用了持续连接(persistent connection)，或者这个信息存储到了缓存或者本地资源上，这个值将和 `PerformanceTiming.fetchStart一致。`

[`PerformanceTiming.connectStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/connectStart) 只读

是一个无符号long long 型的毫秒数，返回HTTP请求开始向服务器发送时的Unix毫秒时间戳。如果使用持久连接（persistent connection），则返回值等同于fetchStart属性的值。

[`PerformanceTiming.connectEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/connectEnd) 只读

是一个无符号long long 型的毫秒数，返回浏览器与服务器之间的连接建立时的Unix毫秒时间戳。如果建立的是持久连接，则返回值等同于fetchStart属性的值。连接建立指的是所有握手和认证过程全部结束。

[`PerformanceTiming.secureConnectionStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/secureConnectionStart) 只读

是一个无符号long long 型的毫秒数，返回浏览器与服务器开始安全链接的握手时的Unix毫秒时间戳。如果当前网页不要求安全连接，则返回0。

[`PerformanceTiming.requestStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/requestStart) 只读

是一个无符号long long 型的毫秒数，返回浏览器向服务器发出HTTP请求时（或开始读取本地缓存时）的Unix毫秒时间戳。

[`PerformanceTiming.responseStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/responseStart) 只读

是一个无符号long long 型的毫秒数，返回浏览器从服务器收到（或从本地缓存读取）第一个字节时的Unix毫秒时间戳。如果传输层在开始请求之后失败并且连接被重开，该属性将会被数制成新的请求的相对应的发起时间。

[`PerformanceTiming.responseEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/responseEnd) 只读

是一个无符号long long 型的毫秒数，返回浏览器从服务器收到（或从本地缓存读取，或从本地资源读取）最后一个字节时（如果在此之前HTTP连接已经关闭，则返回关闭时）的Unix毫秒时间戳。

[`PerformanceTiming.domLoading`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domLoading) 只读

是一个无符号long long 型的毫秒数，返回当前网页DOM结构开始解析时（即[`Document.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/readyState)属性变为“loading”、相应的 `readystatechange`事件触发时）的Unix毫秒时间戳。

[`PerformanceTiming.domInteractive`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domInteractive) 只读

是一个无符号long long 型的毫秒数，返回当前网页DOM结构结束解析、开始加载内嵌资源时（即[`Document.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/readyState)属性变为“interactive”、相应的`readystatechange`事件触发时）的Unix毫秒时间戳。

[`PerformanceTiming.domContentLoadedEventStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domContentLoadedEventStart) 只读

是一个无符号long long 型的毫秒数，返回当解析器发送`DOMContentLoaded` 事件，即所有需要被执行的脚本已经被解析时的Unix毫秒时间戳。

[`PerformanceTiming.domContentLoadedEventEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domContentLoadedEventEnd) 只读

是一个无符号long long 型的毫秒数，返回当所有需要立即执行的脚本已经被执行（不论执行顺序）时的Unix毫秒时间戳。

[`PerformanceTiming.domComplete`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/domComplete) 只读

是一个无符号long long 型的毫秒数，返回当前文档解析完成，即[`Document.readyState`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/readyState) 变为 `'complete'且相对应的``readystatechange` 被触发时的Unix毫秒时间戳。

[`PerformanceTiming.loadEventStart`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/loadEventStart) 只读

是一个无符号long long 型的毫秒数，返回该文档下，`load`事件被发送时的Unix毫秒时间戳。如果这个事件还未被发送，它的值将会是0。

[`PerformanceTiming.loadEventEnd`](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming/loadEventEnd) 只读

是一个无符号long long 型的毫秒数，返回当`load`事件结束，即加载事件完成时的Unix毫秒时间戳。如果这个事件还未被发送，或者尚未完成，它的值将会是0.

:arrow_up_small: 以上属性来自`mdn`文档。



根据上面的定义，可以得出下面的常用的时间计算公式：

**常用计算：**
`DNS`查询耗时 ：`domainLookupEnd` - `domainLookupStart`
`TCP`链接耗时 ：`connectEnd` -` connectStart`
`request`请求耗时 ：`responseEnd` - `responseStart`
解析`dom`树耗时 ： `domComplete` - `domInteractive`
白屏时间 ：`responseStart` -` navigationStart`
`domready`时间(用户可操作时间节点) ：`domContentLoadedEventEnd` -` navigationStart`
`onload`时间(总下载时间) ：`loadEventEnd` - `navigationStart`



#### 参考

[[Performance — 前端性能监控利器](https://www.cnblogs.com/bldxh/p/6857324.html)](https://www.cnblogs.com/bldxh/p/6857324.html)

[w3c文档(<https://w3c.github.io/navigation-timing/#introduction>)](https://w3c.github.io/navigation-timing/#introduction)

[mdn文档(<https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming>)](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceTiming)




---
layout: post
title: processing-小白也可以做艺术家
tags: processing
categories: code
comments: true
abbrlink: e318f551
---
> 对于非计算机专业的人来说，为了做计算机图形而去学习opencv，学习opengl，会是一件很漫长而且艰难的经历，首先你要有最少一门语言基础，c++，python都可以，掌握这些就要花很长的时间，如果你只是想像用草稿纸一样画图，不要去关心一条直线是怎么一个一个像素生成的，也不关心图形学中有多少种曲线的生成方式，你只是想画点可能会很cool的图像，processing很适合你。 

## processing简介
Processing是一种具有革命前瞻性的新兴计算机语言，它的概念是在电子艺术的环境下介绍程序语言，并将电子艺术的概念介绍给程序设计师。它是 Java 语言的延伸，并支持许多现有的 Java 语言架构，不过在语法 (syntax) 上简易许多，并具有许多贴心及人性化的设计。Processing 可以在 Windows、MAC OS X、MAC OS 9 、Linux 等操作系统上使用。目前最新版本为Processing 3。以 Processing 完成的作品可在个人本机端作用，或以Java Applets 的模式外输至网络上发布。 虽然图形用户界面(GUI)早在二十年前成为主流，但是基础编程语言的教学到今天仍是以命令行接口为主，学习编程语言为什么要那么枯燥呢？人脑天生擅长空间辨识，图形用户界面利用的正是这种优势，加上它能提供各种实时且鲜明的图像式反馈 (feedback)，可以大幅缩短学习曲线，并帮助理解抽象逻辑法则。举例来说，计算机屏幕上的一个像素(pixel) 就是一个变量值(the value of a variable) 的可视化表现。Processing将Java的语法简化并将其运算结果“感官化”，让使用者能很快享有声光兼备的交互式多媒体作品。 Processing的源代码是开放的，和近来广受欢迎的Linux 操作系统、Mozilla浏览器、或Perl语言等一样，用户可依照自己的需要自由裁剪出最合适的使用模式。Processing的应用非常丰富，而且它们全部遵守开放源代码的规定，这样的设计大幅增加了整个社群的互动性与学习效率。 以上来自百度百科。

总而言之，学习processing会是一件有趣，不那么艰难的一件事。

[processing的官网](https://processing.org/)

下面贴几张大佬们用processing做的艺术品，希望自己也可以做出这样水平的作品出来。

![img](https://wx3.sinaimg.cn/mw1024/bca3c023gy1g1xw3d8sggj20b4069jse.jpg)

​                                                                                                                          视频中两个酷炫图形过渡

![img](https://wx2.sinaimg.cn/mw1024/bca3c023gy1g1xw3d8nq8j20go0610tx.jpg)

​                                                                                                                     旋转（rotate）、随机（random）

![img](https://wx1.sinaimg.cn/mw1024/bca3c023gy1g1xw3d8kbzj20go09d3ze.jpg)

​                                                                                                                              加上颜色酷炫的随机和旋转

## processing.js
processing.js是一个javascript实现的canvas绘图库,他的作用就是把processing的代码翻译成javascript代码,从而实现在浏览器里面使用processing绘图.  

用法 :
首先下载processing.js
在html代码里创建<canvas>对象,他的data-processing-sources属性指向你写好的.pde代码.
如:
```html
<script src="processing-1.3.6.min.js"></script> 
<canvas data-processing-sources="hello-web.pde"></canvas>
```

最后,canvas里就出现了你在.pde里绘制的图像了。
	
下面介绍`youtube`里面一个教你用`processing`语言的频道  

**The Coding Train**

{% youtube 5N31KNgOO0g %}

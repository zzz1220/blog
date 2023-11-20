---
title: chatgpt辅助代码编写flappy bird 游戏
tags: chatgpt three.js
abbrlink: 6c3f9baf
date: 2023-11-11 14:13:18
---
> 最近我终于下定决心要深入学习 Three.js 这个图形库。然而，我发现官方文档使用的术语有点晦涩，对于初学者来说并不友好。想要了解一些具体的使用方法和含义时，不得不在示例中一点点查找。有时候，想要实现一些效果，也感到无从下手。因此，我在思考，是不是 ChatGPT 能够帮助我解决这些问题呢？

接下来是我两天的使用体验。

# 我问了chatgpt 那些问题？
我一开始是打算做一个简单的flappybird 游戏，中途也遇到了一些实现上的问题，于是尝试向chatpgt 寻找答案。

1. 碰撞检测，一开始我是想直接用 （x，y） 坐标直接判断的，但是有想知道three.js有没有现成的api 可以使用。

问： three.js 中两个矩形，怎么判断是否有重叠部份？

答： 在Three.js中，你可以使用辅助库中的THREE.Box3类来表示矩形（或立方体）的边界框。要检查两个矩形是否有重叠部分，你可以创建两个THREE.Box3实例，分别表示这两个矩形，然后使用intersectsBox方法检查它们是否相交。

下面是一个简单的示例代码：

```javascript
// 创建两个矩形的边界框
var box1 = new THREE.Box3(new THREE.Vector3(x1, y1, 0), new THREE.Vector3(x1 + width1, y1 + height1, 0));
var box2 = new THREE.Box3(new THREE.Vector3(x2, y2, 0), new THREE.Vector3(x2 + width2, y2 + height2, 0));

// 检查两个矩形是否相交
if (box1.intersectsBox(box2)) {
    console.log("矩形相交，存在重叠部分。");
} else {
    console.log("矩形没有重叠部分。");
}

```

问：three.js 中，如何使一个物体实现自由下落的效果，考虑重力加速度为9.8  

答：
在Three.js中实现一个物体自由下落的效果，考虑到重力加速度为9.8，你可以按照以下步骤进行设置：

创建场景（Scene）、相机（Camera）和渲染器（Renderer）：
```javascript

var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```

创建一个球体（Sphere）：

```javascript
var geometry = new THREE.SphereGeometry(1, 32, 32);
var material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
var sphere = new THREE.Mesh(geometry, material);
scene.add(sphere);
设置物体的初始位置和速度：
javascript
Copy code
sphere.position.set(0, 10, 0); // 初始高度
var gravity = 9.8; // 重力加速度
var velocityY = 0; // 初始速度
```

在动画循环中更新物体的位置： 

```javascript
function animate() {
    requestAnimationFrame(animate);

    // 计算新的速度和位置
    velocityY -= gravity * 0.01; // 时间步长为0.01秒
    sphere.position.y += velocityY;

    // 渲染场景
    renderer.render(scene, camera);
}
animate();

```
在这个例子中，物体的垂直速度根据重力加速度逐帧减小，从而实现了自由下落的效果。你可以根据需要调整初始高度、重力加速度和时间步长等参数，以满足你的需求。这是一个简单的演示，如果需要更复杂的物理模拟，可能需要考虑使用更高级的物理引擎。



可以看到，在这种简单场景下，chatgpt 可以做到提出思路和代码，但是更加复杂的场景，chatgpt 却只能给出大概的思路，并且往往是一些正确的废话，不知道是不是我提问的不够详细，还有待观察，有时换了一些关键词，回答就会有很大的变化。


# 结论

作为一个刚刚开始学习 Three.js 的新手，仅仅在阅读文档半天的情况下，通过 ChatGPT 的回答，在空闲时间成功地开发出了一个小游戏。这显示了 ChatGPT 对于普通开发者的巨大帮助，尤其是在遇到不理解的问题时，通过提问可以得到很好的指导。当然，这并不意味着 ChatGPT 的回答始终是准确无误的。在更复杂的领域，特别是需要深入探索和验证的情况下，仍然需要依赖个人的实践和理解。在这里，我只是体验了 ChatGPT 作为一个辅助工具的功能，意识到AI完全替代开发人员还有很长的路要走。

这里贴个demo 地址
https://zzz1220.github.io/threejs-flappybird/


# 是否需要chatgpt4
目前我仍在使用免费的3.5版本，而4.0版本需要每月支付20美元，觉得有些贵。同时，我已经了解了4.0和3.5之间的差异，例如4.0可以将图片作为输入，拥有更强大的上下文理解等功能。然而，考虑到我目前的需求并不涉及这些特性，因此暂时决定不进行升级。

画外音：其实这篇日志，很多内容也是chatgpt 生成的，我感觉我已经离不开它了。
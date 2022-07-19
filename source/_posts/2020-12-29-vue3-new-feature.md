---
title: vue3-new-feature
tags: vue3
abbrlink: 20a24eb5
date: 2020-12-29 15:47:59
---

> 总结一下最近一段时间学习 vue3 的收获

## 功能方面

#### Composition API (组合API)
Composition API 的灵感来自于 React Hooks ，是比 mixin 更强大的存在。它可以提高代码逻辑的可复用性，从而实现与模板的无关性；同时函数式的编程使代码的可压缩性更强。另外，把 Reactivity 模块独立开来，意味着 Vue3.0 的响应式模块可以与其他框架相组合。
官方例子
```javascript
    // 鼠标位置侦听逻辑 
    function useMouse() {
            const state = reactive({
                x: 0,
                y: 0
            })
            const update = e => {
                state.x = e.pageX
                state.y = e.pageY
            }
            onMounted(() => {
                window.addEventListener('mousemove', update)
            })
            onUnmounted(() => {
                window.removeEventListener('mousemove', update)
            })
            return toRefs(state)
        }

```

总的来说，就是把相关的逻辑单独拿出来提高代码复用
#### 全局挂载/配置API更改
vue2 使用 new Vue 创建实例，通过全局设置config配置vue的可选配置。
vue3 使用 createApp 定义的某个 Vue 程序。它可以使你的代码更易于理解，并且不易出现由第三方插件引发的意外问题。目前，如果某些第三方解决方案正在修改 Vue 对象，那么它可能会以意想不到的方式（尤其是全局混合）影响你的程序，而 Vue 3 则没有这个问题.

```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

app.config.ignoredElements = [/^app-/]
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)

app.mount('#app')

```

#### Fragments(片段)
在书写vue2时，由于组件必须只有一个根节点，很多时候会添加一些没有意义的节点用于包裹。Fragment组件就是用于解决这个问题的（这和React中的Fragment组件是一样的）。
#### Suspense
同样的，这和React中的Supense是一样的。Suspense 让你的组件在渲染之前进行“等待”，并在等待时显示 fallback 的内容。
```html
<Suspense>
  <template >
    <Suspended-component />
  </template>
  <template #fallback>
    Loading...
  </template>
</Suspense>

```
####  v-model 支持多个
在 3.x 版本中，在自定义组件上使用 v-model 相当于传递了一个 modelValue 属性以及触发一个 update:modelValue 事件。
如果要改变绑定的属性名，只需要给 v-model 传递一个参数就好了：
```html
<KyrieInput v-model:title="name" />
<!-- 等同于 -->
<KyrieInput :title="name" @update:title="name = $event" />
```
这个写法还彻底代替了 .sync 修饰符，并且支持统一组件绑定多个 v-model

#### Portals
Teleport其实就是React中的Portal。Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。一个 portal 的典型用例是当父组件有 overflow: hidden 或 z-index 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框。
## 实现逻辑重大更改

#### 响应式原理
Object.defineProperty -> Proxy
Object.defineProperty是一个相对比较昂贵的操作，因为它直接操作对象的属性，颗粒度比较小。将它替换为es6的Proxy，在目标对象之上架了一层拦截，代理的是对象而不是对象的属性。这样可以将原本对对象属性的操作变为对整个对象的操作，颗粒度变大。

javascript引擎在解析的时候希望对象的结构越稳定越好，如果对象一直在变，可优化性降低，proxy不需要对原始对象做太多操作。

同时，有了Proxy，就不用考虑新增属性的响应行为了，是时候要跟$set说声再见了。

## 性能方面
双向响应原理由Object.defineProperty改为基于ES6的Proxy，使其颗粒度更大，速度更快，且消除了之前存在的警告；
重写了 Vdom ，突破了 Vdom 的性能瓶颈
进行了模板编译的优化
进行了更加高效的组件初始化


#### 支持tree-shaking , 更加解耦的代码结构
支持了 tree-shaking （剪枝）：像修剪树叶一样把不需要的东西给修剪掉，使 Vue3 的体积更小。

需要的模块才会打入到包里，优化后的 Vue3.0 的打包体积只有原来的一半（13kb）。哪怕把所有的功能都引入进来也只有23kb，依然比 Vue2.x 更小。像 keep-alive 、 transition 甚至 v-for 等功能都可以按需引入。

#### 模板性能提升
在之前的VDOM中，如果msg值发生改变，整个模版中的所有元素都需要重新渲染。但在Vue3.0中，在这个模版编译时，编译器会在动态标签末尾加上 /* Text*/ PatchFlag。只能带patchFlag 的 Node 才被认为是动态的元素，会被追踪属性的修改。并且 PatchFlag 会标识动态的属性类型有哪些，比如这里 的TEXT 表示只有节点中的文字是动态的。
每一个Block中的节点，就算很深，也是直接跟Block一层绑定的，可以直接跳转到动态节点而不需要逐个逐层遍历。
既有VDOM的灵活性，又有性能保证。
vue3 使用 hoistStatic 静态节点提升
当使用hoistStatic时，所有 静态的节点都被提升到render方法之外。这意味着，他们只会在应用启动的时候被创建一次，而后随着每次的渲染被不停的复用。
#### 时间分片
在动态节点和数据的量都很大时，那么在数据更新时，js线程就会用很长的时间来执行vdom的相关计算，如果超过了16ms，造成交互或动画等等卡顿现象。而时间分片就是把vdom的大量计算分成多个小任务，保证每个小任务在16ms内执行完，从而不会阻塞用户交互，避免卡顿现象。
然鹅，此功能还未包含在vue3的更新中。

参考：  
https://zhuanlan.zhihu.com/p/147022323  
https://www.jianshu.com/p/1d2846f2a855  
https://v3.vuejs.org/  
https://segmentfault.com/a/1190000024580501
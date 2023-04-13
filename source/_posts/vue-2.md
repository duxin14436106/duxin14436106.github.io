---
title: vue-2
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: vue-2
date: 2022-12-7 18:18:06
password:
summary:
tags: [vue-2.6.14, 源码]
categories:
---
### 核心地带

在上一篇内容提到的`src/platforms/web/entry-runtime-with-compiler.js`文件中，存在Vue构造函数的引用

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aea9ae7d7f6d424881781be8b4afa6d0~tplv-k3u1fbpfcp-watermark.image?)
现在打开`src/platforms/web/runtime/index.js`可以看到该文件中，大部分都是对于Vue上属性的赋值定义
其中该文件中Vue的来源：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5dd9113af8a48319dc459d6f9c73dd9~tplv-k3u1fbpfcp-watermark.image?)
##### src/platforms/web/runtime/index.js
```
extend(Vue.options.directives, platformDirectives) // platformDirectives: 全局指令model ,show
extend(Vue.options.components, platformComponents) // platformComponents: 全局组件Transition，TransitionGroup
// 但是不太清楚为啥仅仅吧model show 放在这里，后续看一下为啥？？？

Vue.prototype.__patch__ = inBrowser ? patch : noop

Vue.prototype.$mount = function (...) {...}
```


在`src/core/index.js`在该文件中，正式进入到Vue的核心地带，Vue也是从此暴露出去的。【vue的定义是在`src/core/instance/index.js`】,


### core文件夹内容详谈
##### src/core/instance/index.js 中核心内容
```
function Vue (options) {
  ....
  this._init(options)
}

initMixin(Vue) // Vue.prototype._init
stateMixin(Vue) // Vue.prototype.$set,Vue.prototype.$delete, Vue.prototype.$watch,
eventsMixin(Vue) // Vue.prototype.$on,Vue.prototype.$once, Vue.prototype.$off, Vue.prototype.$emit
lifecycleMixin(Vue) // Vue.prototype._update,Vue.prototype.$forceUpdate, Vue.prototype.$destroy, 
renderMixin(Vue) // Vue.prototype.$nextTick, Vue.prototype._render

export default Vue
```
##### src/core/index.js核心代码
```
    import Vue from './instance/index'
    import { initGlobalAPI } from './global-api/index'
    ...
    initGlobalAPI(Vue)
    ...
```
下面探究一下：`initGlobalAPI` 这个方法中干了什么
- 主要是增加了全局的组件，指令方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f85b52eab81d48ba980081927653cf6d~tplv-k3u1fbpfcp-watermark.image?)

## 小总结
写到这个地方，把上面的过程总结一下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f5a7a16e19d4f2788ab45c4fd3a451b~tplv-k3u1fbpfcp-watermark.image?)

# Vue基本代码的执行在源码中的核心流程

像我们前端平常开发Vue项目，已经了解了大部分的Vue的一些技术点：数据双向绑定，监听器，计算属性，生命周期，mixin, 组件传值，模板语法等等
>  以前看过一位大佬的博客 对于源码的总结，很适合我这种初级手了解，现在我根据自己的理解先对代码运行时进行一个简单的小总。
>
**【下图主要是created之前的一个过程】（画的过于潦草，见谅）**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38b38c84bac84d49887600d2a5e07410~tplv-k3u1fbpfcp-watermark.image?)

在代码主要的部分还是init 数据、方法、计算属性，监听器的内容，而这部分出现在initState方法中，下面就继续讨论`initState`方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ed1ba6db82545038c5cb69934fa58e5~tplv-k3u1fbpfcp-watermark.image?)




#### new Dep()：src/core/observer/dep.js
> Dep非常重要，对于响应式，它与要实现响应式功能的数据对象和对象属性关联


使用的实例化场景有两个地方：
-  defineReactive(): url:  `src/core/observer/index.js（135行）`
-  Observer 构造函数内：url: `src/core/observer/index.js(37行)`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebdd9b500650454d825d72f84ece1471~tplv-k3u1fbpfcp-watermark.image?)
> 这样Observer对象与Dep对象一对一关联起来

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/addd1eae7e03452daa2c1695697d9e6d~tplv-k3u1fbpfcp-watermark.image?)
> 此处经过处理后的对象属性具有响应式功能，且每个属性通过必报中保存了一份Dep对象及其他必要信息，最后对象的属性也与Dep对象同样实现了一对一的关联

***Dep源码***
```
export default class Dep {
  static target: ?Watcher; // 静态变量，保存watcher类型对象
  id: number; // 对象的id
  subs: Array<Watcher>; // 订阅者数组，元素即watcher对象

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) { // 添加订阅者（对象的属性）
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) { // 删除订阅者
    remove(this.subs, sub)
  }

  depend () {  // 依赖收集
    if (Dep.target) {
      Dep.target.addDep(this) // addDep（）是watcher构造函数中的方法，用于在subs添加内容
    }
  }
  notify () { // 通知订阅者，更新事件
      const subs = this.subs.slice()
      ...
      for(let i=0, l=subs.length;i<l;i++) {
          sub[i].update()
      }
  }
}

Dep.target = null
const targetStack = []
// watcher 对象
export function pushTarget (target: ?Watcher) { // 入栈
  debugger
  targetStack.push(target)
  Dep.target = target
}

export function popTarget () { // 出栈
  debugger
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}
```
# 后记
掘金同步地址：https://juejin.cn/post/7174351333766463519

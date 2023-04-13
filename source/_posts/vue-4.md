---
title: vue-4
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: vue-4
date: 2023-04-13 11:37:11
password:
summary:
tags: [vue-2.6.14, 源码]
categories:
---

## 回顾
#### 涉及的前端小知识【遇到就记录一下】

在上一篇中提到的`optimize(ast,options)`方法中有一个`markStatic`标识静态节点的方法，在里面有这样的一些代码：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54c3e7b24a3a4d779512006500a12965~tplv-k3u1fbpfcp-watermark.image?)
这个地方涉及到一个[nodeType](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)的小知识点，记录一下：
**Node.nodeType:仅读属性，Node.nodeType表示的是该节点的类型**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e5c4423a4d9472ba0a0d9a0bbf7ec4f~tplv-k3u1fbpfcp-watermark.image?)

1：元素节点  
2：属性节点  
3：文本节点  
4：CDATA区段  
5：实体应用元素  
6：实体  
7：表示处理指令  
8：注释节点  
9：最外层的Root element,包括所有其他节点  
11:文档碎片节点  
12：DTD中声明的符号节点

10：`<!DOCTYPE ...> `


## 全图
跟着`生命周期`，`源码`，`运行过程`描绘完整个全图，大致就是下图这样（真的好丑，但是我并不想重新画一下）

![页面 1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efe255d97af04d5990ebf367bffc95cf~tplv-k3u1fbpfcp-watermark.image?)

# vue中最最最核心的方法
### Watcher源码解析:方法参数的解释：
文件调用路径：(src/core/observer/watcher.js)
- deps[]存放dep的数组
- lazy 标记是否是计算属性watcher
- user标记是否是用户watcher
- getter 渲染函数或表达式
- value计算属性的缓存
- addDep()去重重复dep并记录后调用dep.addSub(this)
- depend()调用所有deps[]中的dep.depend()
- get()主要执行getter()
- update()去重watcher防抖，异步执行run()
- evaluate()缓存计算属性，执行getter
- run()执行get()及侦听器中用户的回调
  **vue文件中使用方式**
```
watch: {
  value: function (value) {
    // update value
    $(this.$el).val(value).trigger('change')
  }
}
computed: { 
    valuse(val){ 
        return val + 'sd'
    } 
}
```
#### render Watcher
**生命周期中调用（src/core/instance/lifecycle.js[197行]）:render Watcher**
```js
new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)// 最后一个参数为true，说明此处调用的是一个render watcher
```
**class Watcher**
```
export default class Watcher {
...
  constructor (
    vm: Component, // vue实例
    expOrFn: string | Function, // 表达式或者函数
    cb: Function, // 回调函数
    options?: ?Object, // 控制选项
    isRenderWatcher?: boolean // 是否是render watcher
  ){
  ...
if (options) {
  this.deep = !!options.deep 
  this.user = !!options.user 标识用户代码，为true时，出现异常时会有错误提示
  this.lazy = !!options.lazy 用于computed watcher,值为true不会执行run()
  this.sync = !!options.sync 用于watch watcher，值为true时，同步执行run(),执行更新
  this.before = options.before // 用于触发beforeUpdate钩子
} else {
  this.deep = this.user = this.lazy = this.sync = false
}
  ...
    this.deps = [] // 执行cleanupDeps后，将newDeps赋值给deps
    this.newDeps = [] 用于手机更新后当前组件实例依赖的deps
    this.depIds = new Set() 执行cleanupDeps后，将newDepIds赋值给depIds
    this.newDepIds = new Set() 防止重复收集依赖
  ...
  
>   这一步的操作就是将`expOrEn`赋值给getter

    if (typeof expOrFn === 'function') {
      this.getter = expOrFn // expOrFn这个参数指的就是：updateComponent函数
    } else {
      this.getter = parsePath(expOrFn)
        ...
    }
    ...
  
  }
```
**watcher中的get方法**
```js
get () {
      pushTarget(this) // 把当前的watcher实例赋值给Dep.target(手机依赖的时候，Dep.target必须有值)
      let value
      const vm = this.vm
      try {
        value = this.getter.call(vm, vm) // 重要，代码发生变化，都是在这一个行执行时发生改变
      } catch (e) {
        ...
      } finally {
        ...
        popTarget() // 将Dep.target的值还原为之前的watcher实例
        this.cleanupDeps() // 这个时候表明watcher的依赖收集完毕
      }
      return value
    }
```
*this.getter打印结果：实际上执行的就是updateComponent*

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f21231f24ce42d09a76879d71600f13~tplv-k3u1fbpfcp-watermark.image?)
```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```
#### _render:执行vue实例的渲染，返回vue实例的vnode(虚拟DOM)

#### _update:执行挂载，内部有个__patch__

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/921ef208491443ba91c22d1518224c54~tplv-k3u1fbpfcp-watermark.image?)


__patch__：src/core/vdom/patch.js----->著名的diff算法(下一篇文章专门记录)

**watcher中的update方法**
```js
update () {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true // 并不会立即执行,也不会加入到异步更新队列
  } else if (this.sync) {
    this.run() // 立即执行
  } else {
    queueWatcher(this) // 将watcher实例加入到异步更新队列中，执行nextTick 等方法，后续单独加一片文章记录一下异步更新机制。
  }
}
```

**watcher中的run方法**
```js
run () {
  if (this.active) { // 标识watcher是否在激活状态下
    const value = this.get() // 执行updateComponent,返回永远是undefined,下面代码不再执行
    if (
      value !== this.value ||
      isObject(value) ||
      this.deep
    ) {
      // set new value
      const oldValue = this.value
      this.value = value
      if (this.user) {
        const info = `callback for watcher "${this.expression}"`
        invokeWithErrorHandling(this.cb, this.vm, [value, oldValue], this.vm, info)
      } else {
        this.cb.call(this.vm, value, oldValue)
      }
    }
  }
}
```
----
## watcher 可以分为三种调用方式
1. computed watcher： 用于计算属性的监听
2. watch watcher： watch监听数据是否变化
3. render watcher :vue渲染
#### 1.computed watcher
initComputed()方法的执行顺序：`beforeCreate--->initComputed()--->created`

-  ...
-  callHook(vm, 'beforeCreate')
-  ...
- initState(vm)[src/core/instance/init.js(57)]
    - if (opts.computed) initComputed(vm, opts.computed)
        - initComputed[src/core/instance/state.js(170)]
- ....
- callHook(vm, 'created')


```js
const computedWatcherOptions = { lazy: true }
function initComputed(){
...
    // 判断用户写法：
        // value(){return 1} 
        // value:{get(){return 1}}
    const getter = typeof userDef === 'function' ? userDef : userDef.get
...
    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions // {lazy:true} 不会立即执行
      )
    }
    if (!(key in vm)) {
      defineComputed(vm, key, userDef) // 定义计算属性
    } else {...}
}
```
#### 2.watch watcher
initWatch()方法的执行顺序：`beforeCreate--->initWatch()[src/core/instance/state.js)(293)]--->created`
```js
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key] // handler 为每个key值对应的方法
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) { // 循环内部每一个key执行creatWatcher
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
    function createWatcher (
      vm: Component,
      expOrFn: string | Function,
      handler: any,
      options?: Object
    ) {
      if (isPlainObject(handler)) {
        options = handler
        handler = handler.handler
      }
      if (typeof handler === 'string') {
        handler = vm[handler]
      }
      return vm.$watch(expOrFn, handler, options) // 内部执行new Watcher 创建实例(src/core/instance/state.js[359])
    }
```
#### 3.render watcher
[render watcher](https://juejin.cn/post/7177646568500101178#heading-6)


# 后记
掘金同步地址：https://juejin.cn/post/7177646568500101178

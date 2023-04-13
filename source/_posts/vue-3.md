---
title: vue-3
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: vue-3
date: 2023-04-13 11:25:25
password:
summary:
tags: [vue-2.6.14, 源码]
categories:
---
# 前言
写完两篇文章后，发现确实了解到了许多以前迷惑的地方，比如`watch 怎么确定写法形式的，Dep是怎么收集的`等等,平常面试经历的还是少，趁着这次多发现一些！！

这篇文章主要记一下生命周期下半部分的执行细节：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3967efa520b7448bbceceb3879b860c0~tplv-k3u1fbpfcp-watermark.image?)
## 回顾
#### 涉及的前端小知识【遇到就记录一下】

Object.create(null)
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/718f121edc2943388f922ac4bc09bfd0~tplv-k3u1fbpfcp-watermark.image?)
##### 为什么vue中使用Object.create()
     主要就是为了防止Object构造函数的原型被修改时对详见的对象产生影响
     因为使用Object.create(null)是干净的，是没有原型的


###### [Object.getPrototypeOf(object)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/GetPrototypeOf)
概念：返回指定对象的原型（内部[[prototype]]属性的值）







![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f858e822208c431aad549a26f105231a~tplv-k3u1fbpfcp-watermark.image?)



















##### 创建对象的方式及区别

|const obj = {}| 打印结果 |
| --- |---|
|const obj = {} |创建
console.log('打印obj', obj)|![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7ff0867e9694323b358d838f44590ac~tplv-k3u1fbpfcp-watermark.image?)
console.log('打印一下obj属性', Object.getPrototypeOf(obj))|![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c31f1e260215429193910700511e2cdf~tplv-k3u1fbpfcp-watermark.image?)
console.log(Object.getPrototypeOf(obj) === Object.prototype)|true

--------

|const obj = Object.create(null)| 打印结果 |
| --- |---|
|const obj = Object.create(null) |创建
console.log('打印obj', obj)|![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c83539e3d0e9484d9776f1e0e9f45abc~tplv-k3u1fbpfcp-watermark.image?)
console.log('打印一下obj属性', Object.getPrototypeOf(obj))|![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99b5b317e89a4500821442d0777cf80c~tplv-k3u1fbpfcp-watermark.image?)
console.log(Object.getPrototypeOf(obj) === Object.prototype)|false
const.log(obj.__proto__)|undefined

**总结**

从上表格得知，两种创建对象的方式，他们的原型对象是不一样的。

Object.create(null)方法创建出来的对象是干净的

若：
```js
const obj = new Object()

Object.create(obj) // 
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80312d7d4c3044cab6d2ce5ea54d5abe~tplv-k3u1fbpfcp-watermark.image?)

## 继续生命周期
> 上一篇文章，自己勾勒了一个created前的一个执行过程及一部分的相关的主要的方法函数

这一部分继续画图：（潦草图，见谅）

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50571f40f1294c56a4e3a2352655e5a5~tplv-k3u1fbpfcp-watermark.image?)


## 在beforeMount、mounted过程中有些重要的过程，单独提出来写上一写：

### getOuterHTML(src/platforms/web/entry-runtime-with-compiler.js)

```js
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
...
  const options = this.$options
  if (!options.render) {
    let template = options.template
    if (template) {
    ...
    } else if (el) {
      template = getOuterHTML(el)
    }
...
  return mount.call(this, el, hydrating)
}

function getOuterHTML (el: Element): string {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    return container.innerHTML
  }
}
```
> 这个位置有一个基础知识点：outerHTML，相对应的有一个innerHTML,现在记录一下两者的区别

**innerHtml**
从对象的起始位置到终止位置的全部内容，*不包括Html标签*

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a010f95f606f4ef1a040e8d0180f449d~tplv-k3u1fbpfcp-watermark.image?)

**outerHtml**
从对象的起始位置到终止位置的全部内容，*包括Html标签*

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11f5fe2527304d5c89bb02627e75b693~tplv-k3u1fbpfcp-watermark.image?)

### compileToFunctions(src/compiler/index.js)
> 主要是做模板编译的操作(模板代码)
```html
<div id="app">
  {{ message }}
  <br>
  <!--  这一段静态节点-->
  {{n}}
  <button @click="()=>{this.n++}">点击+1</button>
</div>
```
>
> parse:会用正则等方式解析template模板中的指令、class、style等数据，形成AST

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ddf07f91f6d4d598f4a5f0af22561dd~tplv-k3u1fbpfcp-watermark.image?)
>
>
> optimize:标记static静态节点（markStatic()，markStaticRoots()）,这里是VUE编译过程得一个优化，在diff算法中，直接跳过静态节点，减少比较的过程，优化了patch的性能

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22ea653d1f6b4cc7b049b727d1f0f68f~tplv-k3u1fbpfcp-watermark.image?)
>
> generate：是将AST树转化成`render function`字符串的过程，得到结果是render字符串以及statucRenderFns字符串【此方法中存在一个genElement方法，内部分别去处理vue中指令：v-for...】


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0675b6918829405a80aa410961096c1c~tplv-k3u1fbpfcp-watermark.image?)


# 后记
掘金同步地址：https://juejin.cn/post/7176577096813133879

---
title: vue-1
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: vue-1
date: 2022-11-23 11:49:00
password:
summary:
tags: [vue-2.6.14, 源码]
categories:
---
# 准备工作：下载源码

首先打开github上[vue的源码](https://github.com/vuejs/vue)，现在使用2.6.x中最新的版本是：2.6.14（2.7.x版本是兼容vue3的特性，这次学习不升级考虑）
### 三部曲

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dd3f03262e24325844ac44ec1f7907e~tplv-k3u1fbpfcp-watermark.image?)
点击`code`,点击`Download ZIP`,点击 `解压缩`，通过`webstorm`打开项目
### 目录结构
利用tree操作生成一个[树结构](https://blog.csdn.net/weixin_40599951/article/details/127727594?spm=1001.2014.3001.5501)（本人记录的一个操作小记录），然后挑挑拣拣，如下：

        D:.
        ├─benchmarks                      性能基准测试
        ├─dist                            打包输出目录
        ├─examples                        测试案例
        ├─flow                            flow语法的类型声明
        ├─packages                        额外的包
        ├─scripts                         配置文件目录
        ├─src                             源码目录（重点）
        │  ├─compiler                     编译器
        │  │  ├─codegen                   吧AST转换成render
        │  │  ├─directives                生成render函数钱处理的
        │  │  └─parser                    解析编译
        │  ├─core                         运行时的核心包
        │  │  ├─components                内部封装的全局组件
        │  │  ├─global-api                内部全局的API
        │  │  ├─instance                  Vue实例相关的
        │  │  ├─observer                  响应式原理
        │  │  ├─util                      工具方法
        │  │  └─vdom                      虚拟dom相关，著名的patch
        │  ├─platforms                    平台相关的编译器代码
        │  ├─server                       服务器渲染相关
        │  ├─sfc                          转换成单文件处理
        │  └─shared                       全局共享的工具常量
        ├─test                            测试目录
        └─types                           TS类型声明
### 启动
打开`package.json`,可以看到`scripts`中dev，现在改动一下：[加了一个--sourcemap]

>  sourcemap: 本质是一个信息文件，存储代码转换前后的对应位置关系，是源代码和生产代码的映射，在开发环境下可以使开发者更方便的调试。但是生产环境不需要配置，以免代码暴露泄露

 ```
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev --sourcemap",
```


     npm run dev //启动vue源码项目

````
 // 接下来可以在exanples新建一个最简单的测试文件：test/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>vue-core-test1</title>
  <script src="../../dist/vue.js"></script>
</head>
<body>
<div id="app">
  {{ message }}
</div>
<script>
  debugger
  new Vue({
    el: '#app',
    data() {
      return {
        message: 'hello world'
      }
    }
  })
</script>
</body>
</html>
````

### 寻找入口文件
 ```
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev --sourcemap",
```
从上面这句话可以分解出几个内容，现在分别解释一下：
- rollup -w -c : rollup是一个javascript模块打包器，-w:rollup的配置，启动监听模式，文件更新会自动打包；-c:指定rollup打包的配置文件
- scripts/config.js : 打包的入口文件
- --environment : 运行的、打包的环境，通过node中的process.env获取
- TARGET:web-full-dev: 作为一个配置，在打包文件中用于参数查找
  `scripts/config.js`
```js
//第三步

const builds = { // 定义一个对象
    ...
    'web-full-dev': {
      entry: resolve('web/entry-runtime-with-compiler.js'),
      dest: resolve('dist/vue.js'),
      format: 'umd',
      env: 'development',
      alias: { he: './entity-decoder' },
      banner
    },
    ...
}
// 第二步
function genConfig (name) {
  const opts = builds[name]
  const config = {
    ...
  }
    ....
  return config
}
// 第一步
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```
> 启动 npm run dev,rollup开始打包，进入`scripts/config.js`，进入第一步，判断TARGET存在，进入第二步，`genConfig('web-full-dev')`,找到第三步中的‘web-full-dev’，entry为入口文件

`src/platforms/web/entry-runtime-with-compiler.js`
可以看到该文件主要是对$mount的一个重新定义，添加了对模板的一些处理办法
- 没有template,根据el中找
- 有template，直接用
- ...

对应的就是我们vue2项目的`main.js`中：

```
new Vue({
 router,
 store,
 render: h => h(App)
}).$mount('#app')
```
### 入口文件解析：entry-runtime-with-compiler.js
```
<script>
  const vm = new Vue({
    el: '#app',
    data() {
      return {
        message: 'hello world'
      }
    }
  })
  vm.$mount('#app')
  console.log(vm.$options.render)
</script>
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6de5ef4d57454baba40d91a89592149b~tplv-k3u1fbpfcp-watermark.image?)

```
// 作用：将VUe的html模板编译成render函数
const mount = Vue.prototype.$mount // $mount 在Vue实例上
Vue.prototype.$mount = function (
  el?: string | Element, // 接收el 容器的id,一般就是‘#app’
  hydrating?: boolean
): Component {
  el = el && query(el)

...
// options: 实例提供的实参比如：{router,store,...}
  const options = this.$options

  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        return this
      }
    } else if (el) {
      template = getOuterHTML(el) // 根据id找到outerHtml,也就是html模板
    }
    if (template) {
        // 进入到此方法，编译html模板生成render
        const { render, staticRenderFns } = compileToFunctions(template, {
          outputSourceRange: process.env.NODE_ENV !== 'production',
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments
        }, this)

      options.render = render // 这样就可以通过vm.$options.render得到上图打印的结果
      options.staticRenderFns = staticRenderFns
    }
  }
  return mount.call(this, el, hydrating)
}
```
> 实例化Vue的时候提供了render,template，el,Vue的优先级是render > template> el

在此过程中`compileToFunctions`这个函数是最重要的
- 将html模板解析成AST
- 对AST优化
- 根据AST生成render函数


# 后记
掘金同步地址：https://juejin.cn/post/7169053532341944334

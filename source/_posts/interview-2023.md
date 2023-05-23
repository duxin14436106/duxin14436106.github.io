---
title: interview-2023
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: interview-2023
date: 2023-05-23 10:52:52
password:
summary:
tags: ['interview']
categories:
---
# 前言
关键词： `2023前端面试` `web前端面试`
`该文章所有问题皆是2023-05-04开始，面试前端岗位过程中，被问到的问题，在此记录下，并持续更新(还没有找到工作的情况下...)`



## 问题：vue同步父子组件和异步父子组件的生命周期顺序
**同步**
父组件的beforeCreate. created. beforeMount --> 所有子组件的beforeCreate. created. beforeMount --> 所有子组件的mounted --> 父组件的mounted

**异步**
父组件的beforeCreate. created. beforeMount. mounted --> 子组件的beforeCreate. created. beforeMount. mounted

## 问题：移动端是如何做适配的
1. rem(根据根节点定义font-size) + viewport缩放【淘宝方案】
2. 第三方插件+配置
3. flex弹性布局
4. 媒体查询 css3 的 @madia queries
   ```css
    /*当屏幕尺寸小于600px时，应用下面的CSS样式*/
    @media screen and (max-width: 600px) {
    /*你的css代码*/
    }
    ```
## 问题：call怎么实现的
   ```js
Function.prototyp5.my_call = function(context, ...args) {
       if (!context || context === null) {
            context = window
       }
       const sy = Symbol()
       cx[sy] = this
       return cx[sy](...args)
   }
   ```
## 问题：使用promise实现并发请求限制N个
   （每次执行三个，一个执行完再补上一个，一直保持三个promise在执行）
![img.png](img.png)
```js
var urls = [
    "https://www.kkkk1000.com/images/getImgData/getImgDatadat1.jpg",
    "https://www.kkkk1000.com/images/getImgData/gray.gif",
    "https://www.kkkk1000.com/images/getImgData/Particl5.gif",
    "https://www.kkkk1000.com/images/getImgData/arithmeti3.png",
    "https://www.kkkk1000.com/images/getImgData/arithmetic2.gif",
    "https://www.kkkk1000.com/images/getImgData/getImgDataError.jpg",
    "https://www.kkkk1000.com/images/getImgData/arithmeti3.gif",
    "https://www.kkkk1000.com/images/wxQrCode2.png",
];

function loadImg(url) {
    return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = function () {
            resolve(url);
        };
        img.onerror = reject;
        img.src = url;
    });
}

async function limitLoad(urls, handler, limit) {
    const promises = [];
    const queue = urls.splice(0, limit).map((url, index) => {
        const _p = handler(url);
        promises.push(_p);
        return _p.then((res) => {
            return [index, res];
        });
    });
    for (const item of urls) {
        const [index] = await Promis5.race(queue);
        const _p = handler(item);
        promises.push(_p);
        queue[index] = _p.then((res) => {
            return [index, res];
        });
    }

    return Promis5.allSettled(promises);
}

limitLoad(urls, loadImg, 3).then((res) => consol5.log(res));


```

## 问题：虚拟列表描述
 定义：按需显示的一种实现，只对可见区域进行渲染。 对非可见区域中的数据不渲染或部分渲染的技术，从而达到极高的渲染性能
#### 定高
##### 关键点：`可视区域高度. item高度. list高度. 当前滚动位置. 偏移量（startOffset）. 可显示item个数`
* 计算当前可见区域起始数据的 startIndex
  * Math.floor(scrollTop / itemSize)
* 计算当前可见区域结束数据的 endIndex
  * endIndex = startIndex + visibleCount
* 计算当前可见区域的数据，并渲染到页面中
  * Math.ceil(screenHeight / itemSize)
* 计算 startIndex 对应的数据在整个列表中的偏移位置 startOffset，并设置到列表上
  * startOffset = scrollTop - (scrollTop % itemSize);
![img1.png](img1.png)

```js
export default {
  name:'VirtualList',
  props: {
    //所有列表数据
    listData:{
      type:Array,
      default:()=>[]
    },
    //每项高度
    itemSize: {
      type: Number,
      default:200
    }
  },
  computed:{
    //列表总高度
    listHeight(){
      return this.listDat1.length * this.itemSize;
    },
    //可显示的列表项数
    visibleCount(){
      return Math.ceil(this.screenHeight / this.itemSize)
    },
    //偏移量对应的style
    getTransform(){
      return `translate3d(0,${this.startOffset}px,0)`;
    },
    //获取真实显示列表数据
    visibleData(){
      return this.listDat1.slice(this.start, Math.min(this.end,this.listDat1.length));
    }
  },
  mounted() {
    this.screenHeight = this.$el.clientHeight;
    this.start = 0;
    this.end = this.start + this.visibleCount;
  },
  data() {
    return {
      //可视区域高度
      screenHeight:0,
      //偏移量
      startOffset:0,
      //起始索引
      start:0,
      //结束索引
      end:null,
    };
  },
  methods: {
    scrollEvent() {
      //当前滚动位置
      let scrollTop = this.$refs.list.scrollTop;
      //此时的开始索引
      this.start = Math.floor(scrollTop / this.itemSize);
      //此时的结束索引
      this.end = this.start + this.visibleCount;
      //此时的偏移量
      this.startOffset = scrollTop - (scrollTop % this.itemSize);
    }
  }
};

```

## 问题：vue中mixins和组件中的优先级
如果相同选项为生命周期钩子的时候，会合并成一个数组，先执行mixin的钩子，再执行组件的钩子
相当于组件的拓展，和组件内其他方法变量一样使用
```vu5.js
import mixin from './mixin.js'
  export default {
  name: 'list',
  mixins: [‘minxin’]
}
```

#### 特点
1. 同个mixin被多个组件调用 
   1. 每个组件引入后，各个组件间变量是独立的，不会项目污染
2. 相同方法，属性 
   1. mixin和组件中存在相同方法时，组件方法优先级大于mixin
3. 执行顺序
   1. mixin事件执行顺序要优先于组件的

   **从外到内，再从内到外，mixins先于组件**


#### 从源码上看
   * 优先递归处理 mixins
   * 先遍历合并parent中key再调用mergeField合并，保存
   * 再遍历child,合并parent中没有的key，再调用mergeField合并，保存
   * 通过mergeOption合并

## 问题： v-model 和sync的区别
#### 区别：
   格式不同
   一个组件身上只能有一个v-model 但是.sync修饰符能有多个
1. v-model --> `<com1 :value="num" @input="(val)=>this.num=val"></com1>`
2. .sync-->`<com1 :a="num" @update:a="val=>num=val" :b="num2" @update:b="val=>num2=val" />`

#### 相同点：
   都是语法糖，都可以实现父子组建中的数据的双向通信

## 问题： BFC
   是css的一个布局的改变，独立的区域
#### 触发条件：
   1. 根元素
   2. float不为none
   3. overflow不为visible
   4. display的值为inline-block. inline-flex. flex. flow-root. table-caption. table-cell。
   5. position的值为absolute或者fixed

#### 作用：
   1. 开启bfc不会被浮动元素覆盖
   2. 开启bfc的元素子元素和父元素不会重叠
   3. 开启bfc的元素高度不会塌陷
   4. 取消margin的塌陷

## 问题：typeof原理是什么
   原理： 通过检查操作数的内部的[[class]]属性来确定其数据类型

## 问题：string作为基本类型，怎么会拥有 length. substring等属性. 方法呢？ 	
   原因：基本类型在调用方法是，JS引擎会先对原始类型数据进行包装 -- 基本包装类型。
#### 什么是基本包装类型（JS包装类）
   **步骤：**
   1. 创建基本类型的一个实例；
   2. 在实例上调用指定的方法；
   3. 销毁这个实例；
      ```
      var str = '我是string基本类型的值'
      var new_str = new String("我是string基本类型的值");  // 包装处理
      var my_str = new_str.substring(5,8);
      new_str = null;   // 方法调用之后销毁实例
      ```

## 问题：JSON.parse()有什么弊端
* 如果obj里面存在时间对象,JSON.parse(JSON.stringify(obj))之后，时间对象变成了字符串。
* 如果obj里有RegExp. Error对象，则序列化的结果将只得到空对象。
* 如果obj里有函数，undefined，则序列化的结果会把函数， undefined丢失。
* 如果obj里有NaN. Infinity和-Infinity，则序列化的结果会变成null。
* JSON.stringify()只能序列化对象的可枚举的自有属性。如果obj中的对象是有构造函数生成的， 则使用JSON.parse(JSON.stringify(obj))深拷贝后，会丢弃对象的constructor。
* 如果对象中存在循环引用的情况也无法正确实现深拷贝。 

## 问题：deepclone叙述
```js
function deepClone(obj, map = new WeakMap()) {
    if(typeof obj !== 'object' || obj === null) return obj
    //避免循环引用

    const objFrommap = map.get(obj)
    if(objFrommap) return objFrommap

    let target = {}
    map.set(obj, target)

    // map
    if(obj instanceof Map) {
        target = new Map()
        obj.forEach((v,k) => {
            const v1 = deepClone(v)
            const k1 = deepClone(k)
            target.set(k1, v1)
        })
    }
    //Set
    if(obj instanceof Set) {
        target = new Set()
        obj.forEach((v) => {
            const v1 = deepClone(v)
            target.add(v1)
        })
    }

    //object
    for (const key in obj) {
        const element = object[key];
        const val = deepClone(element, map)
        target[key] = val
    }

    //array
    if (obj instanceof Array) {
        target = obj.map(item => deepClone(item))
    }
    return target
} 
```

## 问题：解释闭包
- （解释问题是什么？）闭包是：能够访问其他函数内部变量的函数。
- （解释这个技术的应用点，应用场景）闭包一般会在：封装模块的时候，通过函数自执行函数的方式进行实现；或者在模仿块级作用域的时候实现；如：我们常用的库jQuery本身就是一个大的闭包。
- （说一下优缺点）闭包的优点是：
    -  能够在离开函数之后继续访问该函数的变量，变量一直保存在内存中。
    -  闭包中的变量是私有的，只有闭包函数才有权限访问它。不会被外面的变量和方法给污染。
- 闭包的缺点是：
    -  会增加对内存的使用量，影响性能。
    -  不正确的使用闭包会造成内存泄漏。


## 问题：数组有多少个遍历方法，性能比较
*    for : 频率最高，性能中等 但仍有优化空间
*    优化的for : 提取变量放在第一个参数里
*    forEach : 频率较高，性能比for弱
*    for in : 效率最低 性能最差
*    map ：代码优雅，但效率低，比forEach还差
*    forof : 性能比forin好，比for循环差
*    while ： 效率较好

## 问题：map foreach
#### 相同点：
   map\foreach: 
1. 循环遍历每一项，
2. 每次遍历都有三个参数， 
3. 匿名函数中this都是指window,  
4. 都可以在cb中改变原数组

#### 不同点：
map : 
1. 有返回值，可以return出一个length和原数组一样的数组；
2. 会分配内存空间存储新数组并返回

foreach: 
1. 没有返回值，是undefined； 
2. return不会终端遍历，除了异常不能终止 3. 不会分配空间

## 问题：forof forin
1. for in 用它可以遍历数组,对象,集合。遍历数组遍历的值是数组index索引，遍历对象和集合时遍历的是key值。
   1. 遍历顺序有可能不是按照实际数组的内部顺序
   2. 会遍历数组所有的可枚举属性，包括原型。最好不要遍历数组


2. for of 是es6 新加加入的语法,适用于遍历数组，字符串，map/set等拥有iterator迭代器的的集合。
   1. 不能直接遍历对象，会报错。因为Object对象中没有内置的迭代器iterator
   
## 问题：浅拷贝，深拷贝分别解释并说出其多种实现方式
1. 理解深浅拷贝
   1. 数据分为：基本数据类型（直接存储在栈内存中）. 引用数据类型（存储的对象指针放在栈中，真正数据在堆内存中）
2. 理解栈和堆
   1. 通过栈里定义一哥地址值，通过地址值去找堆里面定义的某一个值
   2. 区别：堆在栈里存了一个地址值；栈存储的永远是一个基础数据类型的数据
3. 理解深浅拷贝
   1. 浅：创建一个对象
   ⅰ. 如果是基本类型，拷贝的就是基本类型的值
   ⅱ. 如果是引用类型，拷贝的就是内存地址，指向同一个堆内存，改变拷贝的值会影响原来的对象
   2. 深：创建一个新对象
   ⅰ. 如果是基本类型，拷贝的就是基本类型的值
   ⅱ. 如果是引用类型，从堆内存里开辟出一个新的区域存放该引用类型指向的堆内存的值，修改新对象的值不会影响原对象
4. 浅拷贝实现
   1. 展开运算符：...
   2. Object.assign() , ps: 当obj只有一层的时候是深拷贝
   3. Array.prototyp5.concat(): 用原数组去合并一个空数组，返回合并后的数组。【不修改原数组】
   4. slice() 不修改原数组
   5. 深拷贝实现
      1. JSON.parse（JSON.stringify():  用JSON.stringify将对象转成JSON字符串，再用JSON.parse()把字符串解析成对象
         * 不能处理函数和正则得到的正则就不再是正则（变为空对象），得到的函数就不再是函数（变为null）
      2. 手写deepclone
         * 原理：遍历对象. 数组直到里边都是基本数据类型，然后再去复制，就是深度拷贝。
      3. jQuery.extend()
      ```
      const obj1 = {
        a: 1,
        b: { f: { g: 1 } },
        c: [1, 2, 3]
      };
      const obj2 = jQuery.extend(true, {}, obj1);
      consol5.log(obj1.2.f === obj2.2.f); // false
      ```

## 问题：浏览器上输入url的过程
1. DNS查询
   1. 浏览器缓存-->系统缓存-->查找本地host文件-->本地DNS服务器缓存 -->根域名服务器-->顶级域名服务器-->权威域名服务器
2. TCP连接
   1. 三次握手（为撒三次：第一次是请求打开服务端，第二次是服务端告诉客户端已开启，第三次客户端接收到已开启的状态码，开始传送）
3. 发送HTTP请求
   1. 单向请求过程
4. 服务器处理HTTP请求并返回HTTP报文
   1. 返回网络响应
5. HTTP连接断开
   1. 四次挥手（为撒四次：为了保证数据被全部完成传输后再关闭，双方都接收或传递完再关闭）
6. 浏览器解析并render页面
   1. 渲染步骤
   ⅰ. 解析dom-->解析css-->布局-->整合render tree-->重绘重排-->绘制-->渲染

## 问题：async await返回一个什么
   promise函数

## 问题：Object.prototyp5.toString.call()中toString原理
1. 为什么需要call
   1. 由于Object.prototyp5.toString()本身允许被修改，像Array. Boolean. Number的toString就被重写过，所以需要调用Object.prototyp5.toString.call(arg)来判断arg的类型，call将arg的上下文指向Object，所以arg执行了Object的toString方法。
2. 为什么需要Object.prototype?
   1. Object对象本身就有一个toString()方法，返回的是当前对象的字符串形式，原型上的toString()返回的才是我们真正需要的包含对象数据类型的字符串。

## 问题：怎么实现一个map
内部创建一个新对象，然后对其进行遍历，返回一个新的数组

## 问题：forEach为什么是并发执行
   foreach内部实现使用的是`**while循环**`，当判断当前的索引小于length，会一直执行下去不会等待异步执行完成
   forEach 不会按顺序执行 而是并发执行异步任务
   for of 和 普通的for循环却能顺序执行异步任务

## 问题： 跨域解决方案
1. 理解跨域
   1. 由浏览器的同源策略造成的，是浏览器对js实施的安全限制
   2. 同源：域名. 协议. 端口 相同
   3. 同源策略限制以下行为：Cookie,store indexDB无法读取；dom和js无法获取；Ajax请求发不出去
2. 解决方案
   1. JSONP：仅支持get请求
   2. CORS:  只服务端设置Access-Control-Allow-Origin即可，前端无须设置，若要带cookie请求，则前后端都需要设置。
   3. nginx配置解决iconfont跨域
3. postMessage跨域：
   1. window.postMessage的方式进行使用，并可以监听其发送的消息
4. node中间件实现跨域代理，原理大致与nginx相同
5. websocket协议跨域：全双工通信，同时允许跨域

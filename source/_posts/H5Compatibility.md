---
title: H5Compatibility
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: H5Compatibility
date: 2022-04-17 10:45:37
password:
summary:
tags: [H5兼容性]
categories:
---

## 兼容性
-  禁止复制文本：user-select:none
- 长时间按住页面出现闪退：element{-webkit-touch-callout:none}
- 旋转屏幕，字体调整的问题：标签选择器{-webkit-text-size-ajust:100%}
- tab吸顶的操作，用sticky,部分手机失效，还是要使用切换样式的方式
- 修改placeholder的样式的时候，直接修改颜色无效: 必须要加上opacity: 1
```css
input::-webkit-input-placeholder { /* WebKit, Blink, Edge */
   color:#f6a38d;
   opacity: 1;
}
input:-moz-placeholder { /* Mozilla Firefox 4 to 18 */
   color:#f6a38d;
   opacity: 1;
}
input::-moz-placeholder { /* Mozilla Firefox 19+ */
   color:#f6a38d;
   opacity: 1;
}
input:-ms-input-placeholder { /* Internet Explorer 10-11 */
   color:#f6a38d;
   opacity: 1;
}
```

## babel + css
### babel对ecmascript的支持
**babel将es6、es7、es8等语法转换成浏览器可识别的es5或es3语法。**
babel总共分为三个阶段：解析，转换，生成
（babel从6.0开始，不进行transform,这个过程交给plugin做，所以要配置多个plugin，如果没有配置则经过babel输出的代码是没有改变的。为了解决这个问题，babel提供了预设插件：preset[预设置一组插件来便捷使用这些插件所提供的功能---@babel/preset-env]）

- corejs才是api兼容实现的提供者
- @babel/preset-env: 主要是用来转换已经被正式纳入TC39的语法，还有个作用：对api处理，在代码里引入polyfill。根据浏览器兼容性做针对性的语法转换
- @babel/polyfill是一个运行时包，主要是通过核心依赖core-js@2来完成对应浏览器不支持的新的全局和实例api的添加。
- @babel/runtime：核心思想是以引入替换的方式来解决兼容性问题。是api模拟方案的提供者，是项目生产依赖，而不是开发依赖，安装的时候不要使用-D
- @babel/plugin-transform-runtime就是为了方便@babel/runtime的使用
#### 两种方案
1. polyfill
   a. @babel/preset-env + corejs@3实现语法转换 + 在全局和实例上添加api，支持全量加载和按需加载
   b. 缺点：造成全局污染，且会注入冗余的工具代码
   c. 优点：根据浏览器对新特性的支持度选择性的进行兼容性处理
2. runtime
   a. @babel/preset-env + @babel/runtime-corejs3 + @babel/plugin-transform-runtime实现语法转换 + 模拟替换api，只支持按需加载
   b. 缺点：虽然解决了polyfill的缺点，但是会造成一些不必要的转换，从而增加代码体积

### autoprefixer的样式支持
   css3样式前缀自动补全工具

## ios
1. 滚动条，ios在手动滚动的时候，滚动条会出现，随后消失，安卓没有这个问题。
  **解决：**【加个样式】::-webkit-scrollbar {display: none;}

2. 事件委托：点击事件放在body上或者document委托事件上，点击事件会不生效（场景：全屏弹窗，点击关闭无法关闭）
   **解决：** 委托事件放在非html,body的父级元素了【加了一个全局div】
事件添加到可以点击的标签上

3. iphone及ipad下输入框默认有内阴影
   **解决：** element{-webkit-appearance:none}

4. ios输入单词首字母总是会大写
   **解决：**`<input autocapitalize="off" autocorrect="off" />`

5. ios滚动条有时候会卡顿
   **解决：**-webkit-overflow-scrolling: touch（增加弹性）

6. ios识别长串数字为电话
   **解决：**<meta content='telephone=no' name='format-detection'>

7. 禁止ios弹出各种窗口/长时间按住页面出现闪退
   **解决：**-webkit-touch-callout: none
8. ios缓存问题，从首页点击进入详情页，当通过历史记录返回首页时，整个页面会缓存
   **解决：** 给请求加个时间戳或者重新跳转首页
9. ios下 使用disabled 文字颜色会变得不清晰
   **解决：** input#Stime:disabled { 
                 -webkit-text-fill-color: #;
                 -webkit-opacity: ;
                 color: #;
              }



## 安卓
1. 拨打电话样式不同，ios是出现去拨打电话，安卓是直接跳转到拨号界面
   **解决：** 跳转前，统一加一个弹窗 点击去拨打再去拨打电话

2. 部分型号的安卓手机圆角会失效
   **解决：** background-clip:padding-box

3. 安卓型号太多，有些的手机的分辨率太小，导致有些设备显示图片不清晰
   **解决：** 全部采用图片2倍图显示且background-size: contain

4. 安卓手机下取消语音的输入按钮
   **解决：** input::-webkit-input-speech-button {display: none}

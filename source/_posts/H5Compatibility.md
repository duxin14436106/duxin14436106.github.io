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

## babel + css
### babel对ecmascript的支持
### autoprefixer的样式支持

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
   **解决：**-webkit-overflow-scrolling: touch

6. ios识别长串数字为电话
   **解决：**<meta content='telephone=no' name='format-detection'>

7. 禁止ios弹出各种窗口/长时间按住页面出现闪退
   **解决：**-webkit-touch-callout: none



## 安卓
1. 拨打电话样式不同，ios是出现去拨打电话，安卓是直接跳转到拨号界面
   **解决：** 跳转前，统一加一个弹窗 点击去拨打再去拨打电话

2. 部分型号的安卓手机圆角会失效
   **解决：** background-clip:padding-box

3. 安卓型号太多，有些的手机的分辨率太小，导致有些设备显示图片不清晰
   **解决：** 全部采用图片2倍图显示且background-size: contain

4. 安卓手机下取消语音的输入按钮
   **解决：** input::-webkit-input-speech-button {display: none}

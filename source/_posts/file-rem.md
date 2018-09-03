---
title: 利用CSS3新单位rem实现响应
date: 2016-04-26 22:49:25
tags: CSS
comments: true
categories: 'CSS'
---

做移动端的响应方法有很多，但是我喜欢用 CSS3 的新单位 rem，这个单位非常好用（有个比它还好用的单位 vh，不过兼容性太差，不考虑了），根据不同屏幕，设置不同的基准值，从而实现适配各个屏幕尺寸的移动设备。慕课网有一套非常不错的讲关于 rem 的视频，这里推荐给大家[http://www.imooc.com/learn/494](http://www.imooc.com/learn/494)。
rem----CSS3 中新增的单位，兼容性还不错，常用于移动端实现字体的响应，与 em 不同，rem 根据根元素的 font-size 计算，所以要利用 rem 实现适配各个屏幕的大小，就需要根据不同的屏幕设置根元素不同的 font-size 的值。所以我们需要做下面的一些工作。

### 1.获取浏览器的宽高（对于移动设备就是设备的宽度）

代码如下：

```javascript
var pageWidth = window.innerWidth
var pageHeight = window.innerHeight
if (typeof pageWidth !== 'number') {
  if (document.compatMode === 'CSS1Compat') {
    pageWidt = document.documentElement.clientWidth
    pageHeight = document.documentElement.clientHeight
  } else {
    pageWidt = document.body.clientWidth
    pageHeight = document.body.clientHeight
  }
}
```

<!--more-->

#### 1.1 window.innerWidth 与 document.documentElement.clientWidth

通过测试发现，在 IE9+，chrome，firefox 下利用 window.innerWidth 与 document.documentElement.clientWidth 都可以获取到浏览器的宽高，但是他们有区别：

- window.innerWidth 获取到的宽度是把右侧滚动条<font color="red">算在内</font>的宽度
- document.documentElement.clientWidth 获取到的宽度是把右侧滚动条的宽度<font color="red">不算在内</font>的宽度

但是，window.innerWidth 支持的是 IE9+，到 IE8 以前就会输出 undefined，而 document.documentElement.clientWidth 可以支持 IE7（在 IE 模拟器中，没有 IE6，到 IE5 就只能输出 0 了），在《javascript 高级程序设计》（第三版）中这样写道：“在 IE6 中，这些属性必须在标准模式下才有效；如果是混杂模式，就必须通过 document.body.clientWidth 和 document.body. clientHeight 取得相同信息。”所以 document.body.clientWidth;用于混杂模式，在标准模式下只需要 document.documentElement.clientWidth 即可。

#### 1.2 如何判断浏览器处于什么模式？

Javascript 提供了方法

```javascript
if (document.compatMode === 'CSS1Compat') {
  alert('标准模式')
} else if (document.compatMode === 'BackCompat') {
  alert('混杂模式')
}
```

### 2.计算根元素字体大小

通过上面代码可以拿到浏览器窗口（也就是 document）的宽度，这样就可以计算根元素的基准值了。

假设`1rem = xpx`，这里的`x`可以是任意值，同时这里的`1rem`代表根元素字体大小。

假设在宽度为720px的屏幕上，`1rem = 36px`，即720px的屏幕上根元素字体大小为`36px`，则在360px的屏幕上字体大小为`18px`，因为屏幕宽度和根元素字体大小的比值不变，是20，所以计算公式如下：

```javascript
var fontSize = pageWidth / 20
var html = document.querySelectorAll('html')[0]
html.style.fontSize = fontSize
```

通过以上代码就实现了给根元素设置基准 fontSize。

### 3.将 PSD 中测量出的值换算成 rem

公司设计师给的 PSD 的基准宽度为 720px,所以我在布局的时候常常使用 nexus 5 作为移动端设备去测试，因为它的宽度是 360px，与 720px 刚好是 2 倍的关系，根据这个 2 倍的关系，所以换算步骤如下：

1. PSD 设计图中测量出来的尺寸为 m px，则放在我移动设备中需要写的尺寸为 m/2 px
2. 将 px 换算为 rem，由于在 360px 的移动设备下，font-size 基准值为 360 / 20 = 18px，所以 px=>rem 过程为：rem = m / 2 / 18 = m / 36;

通过以上计算就将 PSD 中的 px 转换为移动设备中的 rem，这样就可以实现对各个移动设备的适配。

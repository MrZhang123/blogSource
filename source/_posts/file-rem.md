---
title: 利用CSS3新单位rem实现响应
date: 2016-04-26 22:49:25
tags: CSS
comments: true
categories: "CSS"
---
做移动端的响应方法有很多，但是我喜欢用CSS3的新单位rem，这个单位非常好用（有个比它还好用的单位vh，不过兼容性太差，不考虑了），根据不同屏幕，设置不同的基准值，从而实现适配各个屏幕尺寸的移动设备。慕课网有一套非常不错的讲关于rem的视频，这里推荐给大家[http://www.imooc.com/learn/494](http://www.imooc.com/learn/494)。
rem----CSS3中新增的单位，兼容性还不错，常用于移动端实现字体的响应，与em不同，rem根据根元素的font-size计算，所以要利用rem实现适配各个屏幕的大小，就需要根据不同的屏幕设置根元素不同的font-size的值。所以我们需要做下面的一些工作。
### 1.获取浏览器的宽高（对于移动设备就是设备的宽度）
代码如下：
```javascript
var pageWidth=window.innerWidth;
var pageHeight=window.innerHeight;
if(typeof pageWidth !=='number'){
	if(document.compatMode==='CSS1Compat'){
		pageWidt=document.documentElement.clientWidth;
		pageHeight=document.documentElement.clientHeight;
	}else{
		pageWidt=document.body.clientWidth;
		pageHeight=document.body.clientHeight;
	}
}
```
<!--more-->
#### 1.1 window.innerWidth与document.documentElement.clientWidth
通过测试发现，在IE9+，chrome，firefox下利用window.innerWidth与document.documentElement.clientWidth都可以获取到浏览器的宽高，但是他们有区别：
- window.innerWidth获取到的宽度是把右侧滚动条<font color="red">算在内</font>的宽度
- document.documentElement.clientWidth获取到的宽度是把右侧滚动条的宽度<font color="red">不算在内</font>的宽度

但是，window.innerWidth支持的是IE9+，到IE8以前就会输出undefined，而document.documentElement.clientWidth可以支持IE7（在IE模拟器中，没有IE6，到IE5就只能输出0了），在《javascript高级程序设计》（第三版）中这样写道：“在 IE6 中，这些属性必须在标准模式下才有效；如果是混杂模式，就必须通过 document.body.clientWidth 和 document.body. clientHeight 取得相同信息。”所以document.body.clientWidth;用于混杂模式，在标准模式下只需要document.documentElement.clientWidth即可。

#### 1.2 如何判断浏览器处于什么模式？
Javascript提供了方法
```javascript
if(document.compatMode==='CSS1Compat'){
    alert('标准模式');
}else if(document.compatMode==='BackCompat'){
    alert('混杂模式');
}
```
### 2.计算rem基准值
通过上面代码可以拿到浏览器窗口（也就是document）的宽度，这样就可以计算根元素的基准值了，计算公式如下：
```javascript
var fontSize=pageWidth / 20;
var html=document.querySelectorAll('html')[0];
html.style.fontSize=fontSize;
```
通过以上代码就实现了给根元素设置基准fontSize。
### 3.将PSD中测量出的值换算成rem
公司设计师给的PSD的基准宽度为720px,所以我在布局的时候常常使用nexus 5作为移动端设备去测试，因为它的宽度是360px，与720px刚好是2倍的关系，根据这个2倍的关系，所以换算步骤如下：

1. PSD设计图中测量出来的尺寸为m px，则放在我移动设备中需要写的尺寸为m/2 px
2. 将px换算为rem，由于在360px的移动设备下，font-size基准值为360 / 20 = 18px，所以px=>rem过程为：rem = m / 2 / 18 = m / 36;

通过以上计算就将PSD中的px转换为移动设备中的rem，这样就可以实现对各个移动设备的适配。

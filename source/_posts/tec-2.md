---
title: H5 Canvas实现图片格式的转换
date: 2016-03-22 23:46:18
tags: canvas
comments: true
categories: "Javascript"
---
&emsp;&emsp;今天早上到公司，看到CTO要求调查关于前端如何实现图片格式转换，自己上网找了下关于canvas如何实现图片格式转换，其实还是蛮简单的，但是因为网上的教程着实写的简单，而且都一样，我也是醉了，所以写下这篇博客，以记录今天的调查结果。
#### 使用Javasript将图片写入画布
```javascript
function convertImageToCanvas(image) {
	var canvas = document.createElement("canvas");
	canvas.width = image.width;
	canvas.height = image.height;
	canvas.getContext("2d").drawImage(image, 0, 0);
	return canvas;
}
```
<!--more-->
#### 用JavaScript将画布保持成图片格式
```javascript
function convertCanvasToImage(canvas) {
	var image = new Image();
    //这里的image后面可以写png，jpeg，但是不能写jpg，写了不会转换的
	image.src = canvas.toDataURL("image/jpeg");
	return image;
}
```
#### 调用函数实现转换
&emsp;&emsp;函数convertImageToCanvas需要传入一个image对象，这个对象需要我们手动去获取，例如，可以在html页面中写入img标签并引入图片，然后利用js获取img对象，将img对象传入该函数，即可获得返回的canvas对象，再将该canvas对象传入函数convertCanvasToImage即可获得返回的image，<font color="red" size=2>但是，这里获得的是base64的编码的图片，本身绘制在canvas画布上的图片右键保存还是png图，所以，要想获得转换后的图片，可以利用a标签在href中写入base64编码获得，点击a标签可以跳转到转换后的图片。</font>关于如将base64编码的图片解码成图片，因为今天下午事儿多，没顾上研究，有时间，研究一下。
实现代码如下：
```javascript
var img=document.getElementById('img');
var body=document.getElementById('body');
var canvas=convertImageToCanvas(img);
convertCanvasToImage(canvas);
var link=document.createElement('a');
body.appendChild(link);
link.innerHTML('跳转图片');
```
&emsp;&emsp;以上代码可以在点击link链接后得到转换后的图片，今天就研究到这些，以后有时间再继续研究完善。
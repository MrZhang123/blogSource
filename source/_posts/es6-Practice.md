---
title: 初探ES6——轮播图实践总结
date: 2016-04-11 00:10:54
tags: Javascript
comments: true
categories: "Javascript"
---
&emsp;&emsp;最近因为同学的一个轮播图不会写，让我萌生了用ES6写一个轮播图的想法（本人喜欢这些玩儿一些新的东西），以前就知道ES6，但是一直没有学，现在终于下决心学了，借助阮一峰老师的书[ECMAScript 6 入门](http://es6.ruanyifeng.com/) 和Youtube上面老外的视频学习ES6非常不错。阮一峰老师的书写的非常详细，推荐想学习的ES6的同学仔细看看。本篇文章会总结在实践中用到的ES6的知识。
## 模板字符串
这是我非常喜欢的ES6的特点之一，非常直观的反应出变量和字符串之间的关系，在ES5中，如果我们想在字符串中添加变量，需要用如下写法：
```javascript
animate(box, 'translate(-' + itemWidth * num + 'px,0)', 1000, function () {
    box.style.transitionDuration = '';
    box.style.transform = 'translate(-800px,0)';
    flag = true;
});
```
现在用ES6的模板字符串，可以直接把字符串和变量相结合，更加易懂。
```javascript
animate(box, `translate(-${itemWidth*num}px,0)`, 1000, function() {
    box.style.transitionDuration = '';
    box.style.transform = `translate(-${itemWidth*(item.length-2)}px,0)`;
    flag = true;
});
```
<!--more-->
是不是非常直观方便，从上面的两个简单示例中可以看出，在ES6中，字符串用反引号（``）标识，这一点需要知道。
还有一个特点，模板字符串可以输出折行的字符串，这在ES5传统字符串中是无法做到的，必须借助（\n），且不能在书写时候写入回车，但是在ES6的字符串模板中，可以直接写入回车，空格，然后在字符串输出时候直接输出，非常方便。
```javascript
let myString=`abc
de
ffff  
fas`;
console.log(myString);
/*输出abc
de
ffff  
fas*/
```
## 对函数的扩展
### 1.给函数设置默认值
在对函数的扩展中，添加了一项给函数设置默认值的功能，这个功能可以说是非常赞的。是否记得我们在ES5中是怎么给函数设置默认值？
```javascript
function test(a,b,c){
    var a=a||10;
    var a=b||15;
    var c=c||20;
    console.log(a+b+c);
}
```
这里我们设置默认值，可以达到自己的预期效果，直到有一天，我们把a=0传入，这时候，我们这么写就不对了，对于程序来说，0就是false，所以a会取默认值10，从而达不到我们预期的效果。但是ES6为我们提供非常好的设置默认值的方式。上面的代码可以改写成下面的这样：
```javascript
function test(a=10,b=15,c=20){
    console.log(a+b+c);
}
```
### 2.箭头函数
了解Coffescript的同学应该清楚，Cofficescript的强大之处在于它的无处不在的箭头函数，写起来非常爽，现在，ES6正式引入箭头函数，让我们的程序可以得到简化，例如：
```javascript
//ES5的写法
var test = function (a,b){
    return a+b;
}
//ES6的箭头函数
var test2 = test(a,b)=>a+b;
```
在写轮播时候，需要鼠标移动到下面的这个小圆点在小圆点类数组对象中是第几个，从而才能让图运动到正确位置，在ES5的时候，我们需要给当前这个对象添加属性，写起来比较繁琐，写法如下：
```javascript
var liList = document.querySelectorAll('li');
for(var i=0;i<liList.length;i++){
    liList[i].index=i;
    liList[i].addEventListener('mouseenter',function(){
        console.log(this.index);
    },false);
}
```
这个this.index属性就是当前的鼠标放上去的元素的索引，然后根据这个索引去得到当前的元素。但是在ES6中，我们可以直接使用箭头函数以及在数组中新引入的findIndex来找到当前的活动元素的索引，代码如下：
```javascript
let liList = document.querySelectorAll('li');
let ArrayliList=Array.form(liList);
for(var i=0;i<liList.length;i++){
    liList[i].index=i;
    liList[i].addEventListener('mouseenter',function(){
        let thisIndex = ArrayliList.findIndex((n) => n == this);
    },false);
}
```
以上代码得到的thisIndex就是当前鼠标放上去的索引，这里我对箭头函数中n这个参数的理解是，传入参数n后会遍历数组中的对象，从而找到与this相等的那个对象，然后返回它的索引，这里用到<font color='red'>Array.from()，这是一个ES6中数组中新增的方法，可以将类数组转换成数组。</font>
## ES6的for...of循环
上面的JS代码循环用了for，其实可以用ES6中的for...of循环去代替，这样写法更加简洁。是否记得JS中的for...in循环，这个循环可以循环键值对中的键，但是无法循环值，而for...of的出现正是为了弥补它的不足，for...of循环可以使用的范围包括数组、Set和Map结构、某些类似数组的对象（比如arguments对象、DOM NodeList对象）、Generator对象，以及字符串。所以我们可以利用该循环替代for循环，但是这里要注意一点<font color='red'>如果直接用for...of循环，在chrome49下会报错，官方已证实这是chrome49的BUG，将会在chrome51中修复</font>，所以我在写的时候，利用Array.from()将NodeList对象转换为数组，这样可以放心操作，代码如下：
```javascript
let liList = document.querySelectorAll('li');
let ArrayliList=Array.form(liList);
for(let li of liList){
    li.addEventListener('mouseenter',function(){
        let thisIndex = ArrayliList.findIndex((n) => n == this);
    },false);
}
```
是不是非常简洁:)
以上就是最近几天对ES6的初探的总结，感觉仅仅只是这些就已经让我感受到ES6的魅力了，接下来，我会再做几个demo，慢慢去熟悉这个新的但是非常好玩儿的ES6。



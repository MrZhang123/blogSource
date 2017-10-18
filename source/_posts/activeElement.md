---
title: H5焦点管理---tabindex与document.activeElement
date: 2016-03-26 10:20:56
tags: HTML5
comments: true
categories: "HTML"
---
&emsp;&emsp;周四下午测试人员提出BUG，说我写的那个类支付宝密码框在按下Tab键时候无法像原生的form表单中的input那样被激活，当时的第一反应是，我的密码框原本就是用div模拟的，怎么可能想form表单一样在Tab键时候激活呢。但是当我打开支付宝官网，按下tab键后，密码框可以被激活，这激起了我的好奇心，也就有了后来对tabindex和document.activeElement的发现。废话不多说，进入正题。
### HTML tabindex属
#### tabindex的设置
&emsp;&emsp;当Tab键用于导航时，tabindex属性规定元素的tab键控制次序，其中tabindex的值为阿拉伯数字，默认情况下越靠前的元素该值越小，所以我们可以通过人为改变tabindex的值来改变按下Tab键后可以被激活元素的激活顺序，只需要做如下设置：
```HTML
<input type="text" tabindex="1">
<input type="text" tabindex="2">
<input type="text" tabindex="4">
<input type="text" tabindex="3">
```
<!--more-->
&emsp;&emsp;以上代码在浏览器中按下Tab键激活顺序就是它们在HTML代码中的顺序，但是我在这里设置第三个input的tabindex比第四个的大，所以按下Tab键后，第四个input框的激活在第三个之前。
#### <font color=red>注意</font>
&emsp;&emsp;1.前面说过tabindex的值越小，越被早激活，但是tabIndex的值要0~32767之间。
&emsp;&emsp;2.如果把tabindex值设置为负值（比如设置为tabindex=-1），则按下Tab键不会激活该元素，但是该元素的focus和blur事件仍然启动。
&emsp;&emsp;3.如果把tabindex设置为0，则带0值tabIndex的元素根据源代码（或默认页面行为）进行排序。
&emsp;&emsp;4.如果页面中出现多个tabindex值相同的元素，则浏览器会把这些元素的tabindex值看做0。
#### 为div与span设置tabindex
&emsp;&emsp;默认情况下按下Tab键可以被选中或者激活的有a,area,button,input,object,select,textarea，但是在现代浏览器和IE9+中给div和span设置tabindex属性也可以被选中，在chrome中，被选中的元素会出现淡蓝色的边框。
### H5焦点管理document.activeElement
&emsp;&emsp;H5添加了辅助管理DOM焦点的document.activeElemnt属性，这个属性始终会引用DOM中当前获得了焦点的元素。元素获取焦点的方式有页面加载，用户输入（通常是通过按Tab键）和代码中调用focus()方法。
&emsp;&emsp;默认情况下，文档刚刚加载完，document.activeElement中保存的是document.body元素的引用。文档加载期间，document.activeElement的值为null。另外顺带提一句，判断是否获得焦点可以使用hasFocus()。
&emsp;&emsp;利用该属性，就可以知道当前哪个元素被Tab键激活，进而就可以获取到被激活元素的各类属性，比如可以在按下Tab键时候去输出当前被激活元素的class属性。代码如下：
```javascript
$(function(){
    var $input=$('input');
    $input.on('keydown',function(e){
        var event=e||window.event;
        if(event.keyCode==9){
            console.log(document.activeElement.attr('class'));
        }
    });
});
```
### 总结
&emsp;&emsp;基于以上两点，通过判断当前被激活的元素的class，就实现了让模拟的密码输入框有和原生的input一样在按下Tab键被激活。

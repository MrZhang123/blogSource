---
title: jquery插件开发
date: 2016-03-21 23:04:53
tags: jQuery
comments: true
categories: "jQuery"
---
由于公司需要写一个功能，而公司的中使用的框架是jquery，所以想到了开发jquery框架。
#### 1.jQuery插件开发方式
jQuery插件开发方式主要有三种：
通过$.extend（），通过$.fn向jQuery添加新的方法，通过$weight（）应用jQueryUI的部件工厂方式创建
#### 2.插件中的this
在插件名字定义的这个函数内部，this指代的是在调用该插件时，用jQuery选择器选中的元素。这里的this已经是jQuery元素，无需再用美元符包装。
#### 3.jQuery链式调用
jQuery支持链式调用，要让插件不打破这个链式调用，只需要return一下，实例如下
```javascript
$.fn.myPlugin = function(){
    //这里面this指的是jquery选中的元素
    this.css('color','red');
    return this.each(function(){
        //对每个元素进行操作
        $(this).append(' '+$(this).attr('href'));
    })
}
```
<!--more-->
#### 4.让插件接受参数
在处理插件参数的接收上，通常用jQuery的extend方法，当给extend传递单个对象时，这个对象会合并到jquery身上，直接可以调用，当给extend方法传递一个以上参数时，它会将所有参数对象合并到
```javascript
$.fn.myPlugin = function(){
    var defaults = {
        'color':'red',
        'fontSize':'12px'
    };
    var setting = $.extend(defaults,options);
    return this.css({
        'color':setting.color,
        'fontSize':setting.fontSize
    });
}

```
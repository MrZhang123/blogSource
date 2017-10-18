---
title: iframe跨域POST提交
date: 2017-01-06 23:07:16
tags: javascript,iframe,POST,form
comments: true
categories: "Javascript"
---
> 以前在面试的时候经常遇到问关于跨域的事儿，所以自己对跨域有一定的概念性了解，知道什么是跨域以及解决跨域的方法，但是具体实际从来没有操作过，直到最近在公司项目中，遇到了一个需要使iframe跨域进行POST提交的实际案例，我才明白具体如何使用iframe进行跨域操作。

<!--more-->

说到跨域，就不得不提起浏览器的同源策略。

同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。

## 源

如果协议，端口（如果指定了一个）和主机对于两个页面是相同的，那么这两个页面就具有相同的源。

从这个定义可以看出，如果两个页面的协议，端口，主机三个只要有一个不一样，就是不同的源，想要相互之间进行交互，就需要进行跨域。

## iframe跨域POST无刷新提交

跨域的方法有很多，像JSONP、iframe、CORS、postMessage等等，由于项目中用到了iframe进行POST跨域，所以本文主要总结一下如何利用iframe进行POST无刷新提交。

我们知道一般提交使用form表单进行提交，但是这种提交会导致页面跳转，所以交互效果不是友好，为了实现无刷新提交，我们会使用Ajax，但是此时可能会出现一个问题----跨域，那么如何解决这个问题呢，可以使用一个隐藏的iframe，我们将要提交的数据提交到这个隐藏的iframe，然后让这个iframe去跳转，这样就可以在视觉上实现页面无跳转刷新（实际上页面还是跳转了，只是iframe被隐藏，我们看不到）。

在提交后我们还要获取到后台给我们返回回来的数据，所以需要在iframe中进行数据的交互同时拿到返回回来的`data`。

* 为了让数据可以顺利的进行数据交互，我们通常使用`document.domain`将域设置到顶级域。
* 为了拿到返回回来的`data`，需要使用一个函数，函数名后台已经告知。

## 附实现代码

```html
<form action="You POST Link" method="post" target="target" id="J_commenting">
    <select name="category" class="select J_filter" id="J_typeFilter">
        <option value="0" selected="selected">Select Category</option>
        <option value="1">Life</option>
        <option value="2">People</option>
        <option value="3">Landscape</option>
        <option value="4">Tech</option>
        <option value="5">Others</option>
    </select>
    <input name="title" type="text" class="misstion-title J_misstion-title">
    <textarea name="desc" class="misstion-description J_description" maxlength="200"></textarea>
    <button class="button J_button" type="submit">Submit</button>
</form>
<iframe name="target" style="display:none;"></iframe>
```

```js
var $button = $('.J_button');
var $commenting = $('#J_commenting');
var $filter = $('.J_filter');
var $misstionTitle = $('.J_misstion-title');
var $description = $('.J_description');


$button.on('click', function () {
    var filterValue = $filter.val();
    var misstionTitleValue = $misstionTitle.val();
    var descriptionValue = $description.val();
    if (filterValue === '0' || misstionTitleValue === '' || descriptionValue === '') {
        alert('Check if you filled out all the fields required');
    } else {
        $commenting.submit();
    }
});

$commenting.on('submit', function () {
    document.domain = 'aa.com';
    window.addData = function (data) {
        var dataCode = data.code;
        var dataMsg = data.message;
        if (dataCode === 0) {
            alert('submit success!');
        } else {
            alert('submit failed!');
        }
    }
});
```
点击提交后，后台返回的数据：

```js
document.domain = "aa.com";
var data = {"code":-2,"info":"please login first","message":"please login first"}; 
if( typeof(parent.window['addData']) == "function"){
    parent.window['addData'](data);
}else if( typeof(window.top['addData']) == "function"){
    window.top['addData'](data);
}
```
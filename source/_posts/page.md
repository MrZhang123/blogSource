---
title: 写一个简单的分页插件
date: 2017-05-07 20:57:48
tags: Javascript
comments: true
categories: "Javascript"
---

前段时间有一个项目需要用一个前端分页，说实话之前自己从来没搞过，这次准备自己搞一下，所以先找了几篇文章看看怎么写，但是网上的文章写分页的都基本看不懂，完全不知所云，所以决定在自己搞定分页后写一下如何弄这个。

<!--more-->

通过看网上的一些例子可以看出，分页基本上都是在html中写一个带id的元素，然后使用js动态添加的html的。

首先确定以下数据

* 总共的数据数——`totalData`
* 每页的要显示的数据——`totalData`
* 当超过多少页后分页中出现...——`minPage`
* 当前后都出现...后，当前页前后有几个分页块——`interval`

确定需要的html结构

```html
<!--1,2,3,4,5,6,7-->
<div class="xm-pagenavi">
    <a class="prev disabled" href="javascript:;">&lt;</a>
    <a class="btn current" href="javascript:;" data-page="1">1</a>
    <a class="btn" href="javascript:;" data-page="2">2</a>
    <a class="btn" href="javascript:;" data-page="3">3</a>
    <a class="btn" href="javascript:;" data-page="4">4</a>
    <a class="btn" href="javascript:;" data-page="5">5</a>
    <a class="btn" href="javascript:;" data-page="6">6</a>
    <a class="btn" href="javascript:;" data-page="7">7</a>
    <a class="next" href="javascript:;">&gt;</a>
</div>
<!--< 1,2,3,4,5,6,7...11 >-->
<div class="xm-pagenavi">
    <a class="prev disabled" href="javascript:;">&lt;</a>
    <a class="btn current" href="javascript:;" data-page="1">1</a>
    <a class="btn" href="javascript:;" data-page="2">2</a>
    <a class="btn" href="javascript:;" data-page="3">3</a>
    <a class="btn" href="javascript:;" data-page="4">4</a>
    <a class="btn" href="javascript:;" data-page="5">5</a>
    <a class="btn" href="javascript:;" data-page="6">6</a>
    <a class="btn" href="javascript:;" data-page="7">7</a>
    <b class="pn-break">...</b>
    <a class="btn" href="javascript:;" data-page="100">100</a
    ><a class="next" href="javascript:;">&gt;</a>
</div>
<!--< 1...4,5,6,7,8...11 >-->
<div class="xm-pagenavi">
    <a class="prev" href="javascript:;">&lt;</a>
    <a class="btn" href="javascript:;" data-page="1">1</a>
    <b class="pn-break">...</b>
    <a class="btn" href="javascript:;" data-page="5">5</a>
    <a class="btn" href="javascript:;" data-page="6">6</a>
    <a class="btn current" href="javascript:;" data-page="7">7</a>
    <a class="btn" href="javascript:;" data-page="8">8</a>
    <a class="btn" href="javascript:;" data-page="9">9</a>
    <b class="pn-break">...</b>
    <a class="btn" href="javascript:;" data-page="100">100</a>
    <a class="next" href="javascript:;">&gt;</a>
</div>
<!--< 1...5,6,7,8,9,10,11 >-->
<div class="xm-pagenavi">
    <a class="prev" href="javascript:;">&lt;</a>
    <a class="btn" href="javascript:;" data-page="1">1</a>
    <b class="pn-break">...</b>
    <a class="btn" href="javascript:;" data-page="94">94</a>
    <a class="btn" href="javascript:;" data-page="95">95</a>
    <a class="btn" href="javascript:;" data-page="96">96</a>
    <a class="btn" href="javascript:;" data-page="97">97</a>
    <a class="btn current" href="javascript:;" data-page="98">98</a>
    <a class="btn" href="javascript:;" data-page="99">99</a>
    <a class="btn" href="javascript:;" data-page="100">100</a>
    <a class="next" href="javascript:;">&gt;</a>
</div>
```

## 初始化页面

根据不同的总页数以及分页块最多有多少个，初始化为两种。

1. 总页数少于分页块最多数时候，此时分页很简单，就是`1,2,3,4,5,6`这类的，此时只需要根据点击的分页块显示出对应的数据即可。

2. 总页数多于分页快儿最多数的时候，此时候分页形式为`< 1,2,3,4,5,6,7...11 >`，此时还需要根据不同的当前页数，变换分页的形式。

所以初始化页面的代码如下：

```JS
function initDraw() {
    let $pagenaviBox = $('<div>').addClass('xm-pagenavi');
    $page.append($pagenaviBox);
    if (pageTotal <= minPage) {
        $('<a>').addClass('prev disabled').attr('href', 'javascript:;').html('<').appendTo($pagenaviBox);
        for (let i = 1; i <= pageTotal; i++) {
            let $btnLink = $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', i).html(i).appendTo($pagenaviBox);
            if (i == 1) {
                $btnLink.addClass('current');
            }
        }
        $('<a>').addClass('next').attr('href', 'javascript:;').html('>').appendTo($pagenaviBox);
    } else {
        draw($pagenaviBox,1);
    }
}
```

## 根据当前页生成不同的html

这里的`draw()`用于根据不同的当前页生成`< 1,2,3,4,5,6,7...11 >，< 1...5,6,7,8,9,10,11 >，< 1...4,5,6,7,8...11 >`这三种形式的分页。

这里需要判断三种分页展示形式的变换临界点(项目中当前页前后分别有前后两页)

* 如果当前页往前三页的分页与第一页的距离为1，则展示为`< 1,2,3,4,5,6,7...11 >`
* 如果当前页往后三页的分页与最后一页的距离位1，则展示为`< 1...5,6,7,8,9,10,11 >`
* 除以上两种情况外的情况展示为`< 1...4,5,6,7,8...11 >`

```js
function draw(pagenaviBox,dataPage) {
    $('<a>').addClass('prev disabled').attr('href', 'javascript:;').html('<').appendTo(pagenaviBox);
    $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', 1).html(1).appendTo(pagenaviBox);
    if (dataPage - 3 <= 1 + 1) {  // < 1,2,3,4,5,6,7...11 >
        for (let i = 2; i <= minPage - 2; i++) {
            let $btn = $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', i).html(i).appendTo(pagenaviBox);
        }
        $('<b>').addClass('pn-break').html('...').appendTo(pagenaviBox);
    } else if (dataPage + 3 >= pageTotal - 1) {// < 1...5,6,7,8,9,10,11 >
        $('<b>').addClass('pn-break').html('...').appendTo(pagenaviBox);
        for (let i = pageTotal - 6; i <= pageTotal - 1; i++) {
            $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', i).html(i).appendTo(pagenaviBox);
        }
    } else {// < 1...4,5,6,7,8...11 >
        $('<b>').addClass('pn-break').html('...').appendTo(pagenaviBox);
        for (let i = dataPage - 2; i <= dataPage + 2; i++) {
            $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', i).html(i).appendTo(pagenaviBox);
        }
        $('<b>').addClass('pn-break').html('...').appendTo(pagenaviBox);
    }
    $('<a>').addClass('btn').attr('href', 'javascript:;').attr('data-page', pageTotal).html(pageTotal).appendTo(pagenaviBox);
    $('<a>').addClass('next').attr('href', 'javascript:;').html('>').appendTo(pagenaviBox);
    $('.btn').removeClass('current');
    $('[data-page="' + dataPage + '"]').addClass('current');
}
```

## 添加点击事件

需要添加点击事件的有两类按钮，一类是分页按钮`btn`，一类是往前往后一页按钮`prev`，`next`按钮。

```js
function btnClick() {
    let $pagenaviBox = $('.xm-pagenavi');
    $pagenaviBox.on('click', function (e) {
        let $target = $(e.target);
        let targetClass = $target.attr('class');
        // 根据不同的class定义不同的事件
        if (targetClass == 'btn') {
            let dataPage = parseInt($target.attr('data-page'));
            if (pageTotal <= minPage) {
                //1,2,3,4,5,6,7,8,9
                $('.btn').removeClass('current');
                $target.addClass('current');
            } else {
                // 重新绘制
                $pagenaviBox.empty();
                draw($pagenaviBox,dataPage);
            }
            let $prev = $('.prev');
            let $next = $('.next');
            $prev.removeClass('disabled');
            $next.removeClass('disabled');
            if (dataPage == 1) {
                $prev.addClass('disabled');
            }
            if (dataPage == pageTotal) {
                $next.addClass('disabled');
            }
        }
        if (targetClass == 'prev') {
            let dataPage = parseInt($('.current').attr('data-page')) - 1;
            if (pageTotal > minPage) {
                $pagenaviBox.empty();
                draw($pagenaviBox,dataPage);
            }else{
                $('.btn').removeClass('current');
                $('[data-page="' + dataPage + '"]').addClass('current');
            }       
            let $prev = $('.prev');
            let $next = $('.next');
            $prev.removeClass('disabled');
            $next.removeClass('disabled');
            
            if (dataPage == 1) {
                $prev.addClass('disabled');
            }
            if (dataPage == pageTotal) {
                $next.addClass('disabled');
            }
        }
        if (targetClass == 'next') {
            let dataPage = parseInt($('.current').attr('data-page')) + 1;
            if (pageTotal > minPage) {
                $pagenaviBox.empty();
                draw($pagenaviBox,dataPage);
            }else{
                $('.btn').removeClass('current');
                $('[data-page="' + dataPage + '"]').addClass('current');
            }
            let $prev = $('.prev');
            let $next = $('.next');
            $prev.removeClass('disabled');
            $next.removeClass('disabled');
            if (dataPage == 1) {
                $prev.addClass('disabled');
            }
            if (dataPage == pageTotal) {
                $next.addClass('disabled');
            }
        }
    });
}
```

## 总结

经过这三步，基本就完成了一个非常简单的分页，在我开始自己写之前参考也曾经看了一些比较好的jquery插件的源码，但是感觉并没有找到思路，反而越看越乱，所以说呢…需要写一个东西，直接上手写是最好的，边写边想即可，如果一味的希望先找到思路，想好各个步骤再写，反正对我来讲，会让我越想越混乱。

最后附代码地址：https://github.com/MrZhang123/Js_Plugin/tree/master/page
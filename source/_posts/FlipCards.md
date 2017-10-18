---
title: CSS3实现翻转卡牌效果
date: 2016-08-17 23:35:52
tags: CSS
comments: true
categories: "CSS"
---
> 今天在工作中遇到需求，需要用CSS3写一个卡片翻转效果，这个效果看起来简单，但是还是涉及到一些不常用的CSS3中的3D转换的属性以及实现该效果的思路，所以这里总结一下。

项目地址：[https://github.com/MrZhang123/Practice/tree/master/FlipCards](https://github.com/MrZhang123/Practice/tree/master/FlipCards)
<!--more-->
## 需要的CSS3属性

### perspective

**目前浏览器都不支持`perspective`属性**

在chrome和Safari中需要使用`-webkit-perspective`，而在Firefox中需要使用`-moz-perspective`来定义。

#### 定义和用法

`perspective`属性定义3D元素距视图的距离，以像素计。该属性允许改变查看3D元素的视图。当为元素定义

`perspective`属时，其子元素会获得透视效果，而不是元素本身。换句话说，设置这个元素是为了给该元素的子元素用。

#### 值

number：元素距离视图的距离，以像素计

none：**默认值**，与0相同。不设置透视

### transform-style

Firefox支持`transfrom-style`属性。

Chrome、Safari和Opera支持代替的`-webkit-transform-style`属性。

#### 定义和用法

`transform-style`属性固定如何在3D空间中呈现被嵌套的元素。一般给父元素设置让其所有子元素跟着父元素一起位置移动，一般会设置。

#### 值

flat：子元素将不保留其3D位置（默认值）

perserve-3d：子元素将保留其3D位置

### transition

IE10+、Firefox、Opera、Chrome均支持`transition`属性。Safari支持替代的`-webkit-transition`属性。但是IE9-不支持该属性。

#### 定义和用法

`transition`属性是一个简写属性，用于设置四个过渡属性：

- transiton-property：规定设置过渡效果的CSS属性的名称
- transiton-duration：规定完成过渡效果需要多少秒或毫秒
- transiton-timing-funciton：规定速度效果的速度曲线
- transition-delay：规定过渡效果何时开始

**`transiton-duration`必须设置，否则时长为0 ，不会有过渡效果**

### backface-visibility

只有IE10+和Firefox支持`backface-visibility`，Opera15+、Safari和Chrome支持替代的`-webkit-backface-visibility`

#### 定义和用法

`backface-visibility`属性定义当前元素不面向屏幕时是否可见，如果元素在旋转后不希望看到背面，则可以使用。

visible：背面是可见的（默认值）

hidden：背面是不可见的

## 实现的思路

要实现类似的翻牌效果，首先我们需要有一张可以翻的牌，这张牌由两个元素重叠组成，位于上层正面我们看到，而位于下层的背面我们看不到并且本身是绕Y轴旋转过的，这样，当鼠标移动上去后，同时让正面和背面执行旋转就可以实现翻牌效果。

## 实现

基本结构代码如下：

```html
<div id="content">
    <ul>
        <li>
            <a href="#" >
                <div>
                    <h3>测试正面1</h3>
                    <p>文字文字文字文字文字文字文字文字文字文字文字</p>
                </div>
                <div>
                    <h3>测试背面1</h3>
                    <p>文字文字文字文字文字文字文字文字文字文字文字</p>
                </div>
            </a>
        </li>        
    </ul>
</div>
```

```css
ul,li {
    margin:0;
    padding:0;
    list-style:none;
}
#content ul li{
    width: 225px;
    height: 180px;
}
#content ul li a{
    position: relative;
    display: block;
    width: 100%;
    height: 100%;
}
#content ul li a > div{
    position: absolute;
    left: 0;
    height: 0;
    width: 100%;
    height: 100%;
    color: #fff;
}
#content ul li a div:first-child{
    background-color: rgb(255, 64, 129);
    z-index:2;
}
#content ul li a div:last-child{
    background:rgb(0, 188, 212);
    z-index:1;
}
#content ul li a div h3{
    margin: 0 auto 15px;
    padding: 15px 0;
    width: 200px;
    height: 16px;
    line-height: 16px;
    font-size: 14px;
    text-align: center;
    border-bottom: 1px #fff dashed;
}

#content ul li a div p{
    padding: 0 10px;
    font-size: 12px;
    text-indent: 2em;
    line-height: 18px;
}
```

这样就让两个div叠在一起了，但是如果要进行翻转的话，首先是需要将背面本身就翻过去，当鼠标放上去之后翻转过来，让我们看到，所以我们需要给背面添加翻转180°的属性，鼠标放上去之后让它翻转到0°，同时为保证每个浏览器渲染统一，给正面加一个翻转0°，鼠标移动上去之后翻转-180°，并且是一个动画，所以要添加过渡。所以给正面背面添加CSS如下： 

```css
#content ul li a > div{
    -webkit-transition:.8s ease-in-out;
    -moz-transition:.8s ease-in-out;
}
#content ul li a div:first-child{
    -webkit-transform:rotateY(0);
    -moz-transform:rotateY(0);
}
#content ul li a div:last-child{
    -webkit-transform:rotateY(180deg);
    -moz-transform:rotateY(180deg);
}
#content ul li a:hover div:first-child{
    -webkit-transform:rotateY(-180deg);
    -moz-transform:rotateY(-180deg);
}
#content ul li a:hover div:last-child{
    -webkit-transform:rotateY(0);
    -moz-transform:rotateY(0);
}
```

在添加这些CSS3属性后，可以实现翻转，但是发现只看到正面，没有看到背面，这又是为什么呢，前面提到有一个属性`backface-visibility`，这个属性用于控制在翻转后，元素的背面是否可见，默认是可见的，所以就会挡着背面那个元素，我们需要手动设置元素翻转后背面不可见，所以需要设置：

```css
#content ul li a > div{
     -webkit-backface-visibility: hidden;
     -moz-backface-visibility: hidden;
}
```

这样设置之后，由于正面的元素在翻转后背面不可见，我们就可以看到背面的元素了。

但是，仔细观察会发现这个翻转似乎并不是那么立体，似乎在两条平行线之间实现了翻转，所以我们需要设置一个观察点距离视图的距离，这时候就需要给父元素添加`perspective`属性，这个属性的值一般为800px ~ 1000px，这个范围内的值会看上去合理。所以给父元素添加：

```css
#content ul li a{
    -webkit-perspective: 800px;
    -moz-perspective: 800px;
}
```

至此，就实现了一个翻转卡牌的效果，但是这里需要解决一个问题，因为`perspective`属性不被IE支持（Microsoft Edge支持），所以需要进行降级，我的做法是直接显示隐藏。那么如何识别IE9+浏览器呢？在[stackoverflow](http://stackoverflow.com/)中我找到了答案：

## 附：CSS中识别各类IE的方法

## IE6

```css
* html .ie6{
    property:value;
}
```
or

```css
html .ie6{
    _property:value;
}
```
## IE7

```css
*+html .ie7{
    property:value;
}
```

or

```css
*:first-child+html ie7{
    property:value;
}
```

## IE6 and IE7

```css
@media screen\9{
    ie67{property:value;}
}
```

or

```css
.ie67{ *property:value;}
```
or

```css
.ie67{ #property:value;}
```

## IE6,7 and 8

```css
@media \0screen\,screen\9{
    ie678{property:value;}
}
```

## IE8

```css
html>/**/body .ie8{property:value;}
```

or

```css
@media \0screen{
    ie8{property:value;}
}
```

## 只在IE8标准模式

```css
.ie8{property/*\**/:value\9;}
```

## IE8,9 and 10

```css
@media screen\0{
    ie8910{property:value;}
}
```

## IE9 only

```css
@media screen and (min-width:0\0) and (min-resolution: .001dpcm){
    /*IE9 CSS*/
    .ie9{property:value;}
}
```

## IE9+

```css
@media screen and (min-width:0\0) and (min-resolution: +72dpi){
    /*IE9+ CSS*/
    .ie9up{property:value;}
}
```

## IE9 and 10

```css
@media screen and (min-width:0){
    .ie910{property:value;}
}
```

## IE10 only

```css
_:-ms-lang(x), ie10 {property:value;}
```

## IE10+

```css
_:-ms-lang(x), ie10up{property:value;}
```

or

```css
@media all and (-ms-high-contrast:none),(-ms-high-contrast:active){
    .ie10up{property:value;}
}
```

## IE11+

```css
_:-ms-fullscreen, :root .ie11up{property:value;}
```
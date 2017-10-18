---
title: 调用Google Map Api实现自定义Google Map
date: 2017-06-11 00:10:54
tags: Javascript
comments: true
categories: "Javascript"
---

>前段时间做了一个简单调用Google Map API的项目，因为之前没有接触过这些，所以在看了文档做出来之后自己稍微总结一下。

<!--more-->

## 加载 Google Maps JavaScript API

官方指导教程中写的引入的Google Map API是异步调用的，引入方式如下：

```html
<script>
  var map;
  function initMap() {
    map = new google.maps.Map(document.getElementById('map'), {
      center: {lat: -34.397, lng: 150.644},
      zoom: 8
    });
  }
</script>
<script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap"></script>
```

这里需要注意一点官方提供是异步加载Google Maps JavaScript API，这样会让网站运行速率更高，`但是这样会有一个问题，就是在自己写的js必须与引用在引入api之前，否则会报错。`由于项目中js api只能在自己写的js的后面，所以只能使用同步的方式引入api js，引入方式如下：

```js
<script src="https://maps.googleapis.com/maps/api/js?key=yourkey"></script>
```

这里的key是通过申请成为谷歌开发者然后获取的密钥。

## 创建地图对象

地图的 JavaScript 类是 Map 类。该类的对象定义页面上的单个地图。（创建该类的多个实例–每个对象都将定义页面上的一个不同地图。）我们利用 JavaScript new 运算符来新建该类。

```js
//初次进入页面只显示首都不显示坐标，只定义地图
const map = new google.maps.Map(document.getElementById('map'), {
  center: capitalLatLng,
  zoom: mapZoom,
  maxZoom: mapMaxZoom,
  minZoom: mapMinZoom,
  //禁用街景地图
  streetViewControl:false,
  mapTypeId: google.maps.MapTypeId.ROADMAP,
  //控制地图类型控件的位置
  mapTypeControlOptions: {
    style: google.maps.MapTypeControlStyle.HORIZONTAL_BAR,
    position: google.maps.ControlPosition.TOP_RIGHT
  },
});
```

- center：设置地图初始化后中心点的位置，接受经纬度对象{lat: -25.363, lng: 131.044}
- zoom：地图初始化后的缩放比例
- mapTypeId：设置地图类型，这里设置的就是普通的街道地图
- mapTypeControlOptions：修改地图类型控件

谷歌地图默认会显示缩放控件（zoomControl），地图类型控件（mapTypeControl），街景小人（streetViewControl），全屏按钮（fullscreenControl，移动设备可见），除此之外还有旋转控件（rotateControl，默认情况下，该控件是否显示取决于给定地图类型在当前的缩放级别和位置上是否存在 45° 图像），比例控件（scaleControl，默认不显示）。但是有些时候可能需要调节他们的位置或者是否显示，可以通过相应的Options去控制，例如在上面初始化中，控制地图类型控件的位置。

## 创建地图marker

我们在创建好地图后需要根据坐标去在地图中标注出相应的位置（marker对象），创建方式如下：

```js
//latlngArr为所地理位置数组
latlngArr.map((position, index) => {
  let marker = new google.maps.Marker({
    position,
    zIndex: 0,
    map
  });
});
```

其中`map`为前面所创建的地图，`postion`则为marker的坐标位置。

#### 给marker添加事件

经常见到在点击marker出现弹窗，这个效果可以通过给marker添加点击事件

```js
marker.addListener('click', function (e) {
  map.setZoom(mapMaxZoom);
});
```

这里实现了一个点击设置地图放大到最大的效果，这里需要多说一句，就是在对象中的熟悉是可以通过`set`去设置的。

#### *设置marker自动居中

当设置了多个marker在地图中后，会发现点击某个不在中心的marker后，它并不会自动居中，这一点很不友好，为了设置点击自动居中，需要添加如下代码：

```js
//map auto center
const bounds = new google.maps.LatLngBounds();
latlngArr.map((position, index) => {
  let marker = new google.maps.Marker({
    position,
    icon: notSelect,
    zIndex: 0,
    map
  });
  bounds.extend(marker.position);
});
map.fitBounds(bounds);
map.panToBounds(bounds);
```

- 一个`LatLngBounds`实例表示地理坐标中的矩形，包括穿过180度纵向子午线在内的矩形
- `LatLngBounds`类下的`extend`方法用于扩展这个边界以包含给定的点（marker）
- `fitBounds`设置视口包含给定的边界

#### *设置地图缩放自动调整

根据marker的分布自动调整地图缩放比例是很常见的交互，Google Map API提供了`panToBounds`方法用于根据给定的`LatLngBounds`绘制出包含最小边界的地图，所以在代码中只需要设置`map.panToBounds(bounds)`即可。

## 总结

真心觉得Google的文档写的很好，而且Google Map的功能很强大，我在项目中只是使用了些简单的功能，更复杂功能请参考Google Map API：https://developers.google.com/maps/documentation/javascript/reference?csw=1
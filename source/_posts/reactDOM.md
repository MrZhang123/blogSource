---
title: 小记：React操作真实DOM实现动态吸底部
date: 2017-10-22 17:23:11
tags: Javascript
comments: true
categories: "Javascript" 
---

动态吸底：开始时fixed在页面上，当页面滚动到距离底部一定距离的时fixed部分固定。

<!--more-->

这个是需要计算页面滚动距离的，如果使用Jquery或者原生js实现是非常好实现的，但是由于使用react并不推崇操作DOM，但是如果使用virtual DOM的话是无法实现该效果的，所以还是要引入js去直接获取DOM进行操作。

react在`componentDidMount`之后页面渲染完成，所以可以在这里面直接用js原生方法获取DOM元素，进而进行操作。

```js
componentDidMount() {
  this.changeFixed()
}

//計算高度
changeFixed(){
  //getDOMNode
  const layoutNode = document.querySelectorAll('.page-layout')[0];
  const orderPriceNode = document.querySelectorAll('.test-price')[0];

  window.addEventListener('scroll', function (e) {
    const windowInnerHeight = window.innerHeight;
    const layoutNodeHeight = layoutNode.offsetHeight;
    //滚动超出视野距离
    let scrollTop = window.pageYOffset|| document.documentElement.scrollTop || document.body.scrollTop;
    const distanceBottom = layoutNodeHeight - scrollTop - windowInnerHeight;
    //120的时候吸底
    if(distanceBottom <= 120){
      orderPriceNode.classList.remove('fixed');
    }else{
      orderPriceNode.classList.add('fixed');
    }
  })
}
```

这样就实现了当距离底部120的时候吸底
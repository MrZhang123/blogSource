---
title: 基于Vue的简单的单页面应用
date: 2016-06-08 01:14:39
tags: vue
comments: true
categories: "Vue"
---
## 基于Vue的简单的单页面应用
在对Vue和webpack有了一定了解后，我们就可以开始利用所了解的东西做一个简单的webapp了，不了解的同学可以看下我的前两篇关于vue和webpack的基本应用：
[webpack+vue起步](https://segmentfault.com/a/1190000005614864)
[利用webpack和vue实现组件化](https://segmentfault.com/a/1190000005616974)
<!--more-->
## 构建项目
首先创建各个组件，我的目录结构如下：
```js
//没有后缀名的都是文件夹
|-wechat
    |-dist
    |-src
    |  |-components         //存放vue组件
    |  |    |-tab           //存放home.vue中的tab，动态切换的模板
    |  |    |    |-tab_1.vue
    |  |    |    |-tab_2.vue
    |  |    |-home.vue      //app的首页
    |  |    |-list.vue      //点击home中的链接跳转到
    |  |    |-detail.vue    //点击list中的链接跳转到
    |  |-app.vue            //主要的vue文件(用于将各个组件的挂载)
    |  |-main.js            //主要的js(用于配置路由)    
    |-static                //存放静态资源
    |-index.html
```
## 配置路由
首先在我们的项目中安装vue-router
```js
npm install vue-router
```
引入各个组件并配置路由：
```js
//main.js

import Vue from 'vue';
import VueRouter from 'vue-router';
//引入组件
import App from './app.vue';
import home from './components/home.vue';
import list from './components/list.vue';
import detail from './components/detail.vue';

Vue.use(VueRouter);

var app=Vue.extend(App);

var router=new VueRouter();

//配置路由
router.map({
  '/home': {
    component: home
  },
  '/list': {
    component: list    
  },
  '/detail': {
    component: detail
  }
});
//设置默认情况下打开的页面
router.redirect({
  '/':'home'
});
router.start(app,'#app');
//暴露路由接口调试
window.router = router;
```
关于vue-router的介绍，官方文档介绍很清楚，地址：[http://router.vuejs.org/zh-cn/index.html](http://router.vuejs.org/zh-cn/index.html)。
配置好路由后，需要将匹配好的组件正确的渲染到页面中，此时用到`<router-view></router-view>`，它基于Vue的动态组件系统，所以它会继承一个正常动态组件的很多特性。在这里我们用到两个：
- `v-transition`和`transition-mode`的完整支持，为了切换效果能正常工作，路由组件必须不是一个[片段实例](http://vuejs.org/guide/components.html#Fragment_Instance)。
- 在路由的0.7.2+中支持`keep-alive`（[关于keep-alive](https://vuejs.org.cn/guide/components.html#keep-alive)）

所以在app.vue写入：
```html
<template>
	<div class="main">
		<router-view
	      keep-alive
	      transition="fade"
	      transition-mode='out-in'></router-view>
	</div>
</template>
```
打开命令行启动webpack-dev-server：
```js
$ webpack-dev-server --inline --hot
```
此时我们在页面中看到的页面就是home.vue
## 在home.vue中实现tab切换
tab切换作为一个常见的效果，出现的频率很高，那么如何用vuejs写一个tab切换效果呢？
利用当前被点击的tab是第几个，从而动态的切换相应的动态组件是vuejs实现切换的一种方式。动态组件的介绍如下：[https://vuejs.org.cn/guide/components.html#动态组件](https://vuejs.org.cn/guide/components.html#动态组件)。所以实现代码如下：
```html
<template>
<div class="home">
    <div class="bd" style="height: 100%;">
        <div class="weui_tab">
            <ul class="weui_navbar">
                <li class="weui_navbar_item"
                    v-for="tab in tabs"
                    :class="{'weui_bar_item_on':$index===selected}"
                    @click="choose($index)">{{tab.tabName}}</li>
            </ul>
            <div class="weui_tab_bd">
                <component :is="currentView" transition="fade" transition-mode="out-in"></component>
            </div>
        </div>
    </div>      
</div>
</template>
<script>
import  tab_1 from './tab/tab_1.vue';
import  tab_2 from './tab/tab_2.vue';
export default{
    data(){
        return{
            tabs:[
                {tabName:'Vuejs'},
                {tabName:'VueTab'}
            ],
            selected:0,
            currentView:'view_0'
        }
    },
    components:{
        'view_0':tab_1,
        'view_1':tab_2
    },
    methods:{
        choose(index) {
            this.selected=index;
            this.currentView='view_'+index;
        }
    }
}
</script>
```
两个动态组件为tab_1.vue和tab_2.vue。引入这两个模块，对外输出对组件的操作`export default{}`，在`template`模板中将动态组件加载进去，使用保留的`<component>`元素，动态地绑定它的`is`特性，从而根据不同的值动态的切换组件，在需要点击的tab导航上，需要`v-for`循环出两个导航，然后动态绑定class，根据当前点击的tab导航`$index`动态的切换class名`:class="{'weui_bar_item_on':$index===selected}"`，然后给`li`绑定click事件，从而让其在被点击时执行事件`@click="choose($index)"`。
由于默认情况下显示第一个组件且第一个`tab`变灰，所以在`data`设置默认值。为了切换有过渡，添加了`transition="fade" transition-mode="out-in"`并在css中设置动画的执行过程：
```css
/*切换动画*/
.fade-transition {
    transition: opacity 0.3s ease;
}
.fade-enter,
.fade-leave {
    opacity: 0;
}
```
## 利用`v-link`实现路由链接
在组件中，用到了路由，在给`a`写路由链接时候要使用`v-link`而不是`href`。在带有`v-link`指令的元素，如果`v-link`对应的URL匹配当前路径，则该元素会被添加一个特定的class，默认为`.v-link-active`，这个默认值，我们可以通过在创建路由时指定`linkActiveClass`全局选项来自定义，也可以通过`activeClass`内联选项来单独制定：
```html
<a v-link="{path:'/a',activeClass:'active'}">test</a>
```
## 遇到的一些问题
### 1.`v-for`循环插入图片
在写循环插入图片的时候，写的代码如下：
```html
<div class="bio-slide" v-for="item in items">   
    <img src="{{item.image}}">
</div>
```
此时在控制台会出现警告
`[Vue Warn]: src="{{item.image}}": interpolation in "src" attribute will cause a 404 request. Use v-bind:src instead.`
这里意思是`在src属性插值将导致404请求。使用v-绑定：src代替。`
所以替换成如下：
```html
<div class="bio-slide" v-for="item in items">   
    <img v-bind:src="item.image">
</div>
```
这里需要主要，v-bind在写的时候不建议再用双花括号，根据官方的说法：
```html
<a v-bind:href="url"></a>
```
这里` href `是参数，它告诉 `v-bind `指令将元素的 `href `特性跟表达式 url 的值绑定。可能你已注意到可以用特性插值` href="{{url}}"` 获得同样的结果：这样没错，并且实际上在内部特性插值会转为` v-bind` 绑定。
### 2.`v-model`的使用
`v-model`用于在表单上创建双向绑定，只能用于`<input>`、`<select>`、`<textarea>`，如果用在其他元素中，则会在产生警告。
### 3.如何让组件的CSS样式只在组件中起作用
在每一个vue组件中都可以定义各自的css，js，如果想写的css只对当前组件起作用，则在`style`中写入`scoped`，即：
```css
<style scoped></style>
```
这样就完成了一个简单的基于Vue+webpack+vue-router的单页面应用，具体实现代码见github:[vue_spa_demo](https://github.com/MrZhang123/Vue_project/tree/master/vue_spa_demo)。
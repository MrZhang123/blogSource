---
title: Vuejs的一些总结
date: 2016-07-14 00:07:45
tags: vue
comments: true
categories: "Vue"
---
> 最近一段时间忙着工作的一些事情，同时自己也在试着把项目中的一些移动端页面试着用vuejs重写，所以没时间写文章，今天终于有空可以写一下，由于页面并没有写完，所以就将自己这几天做页面的时候遇到的一些问题总结了一下。其实很多在官网都能找到，但是我们只看官网教程不去写，很难理解到底是什么意思，这里我把我用到的列出来。

文章中提到的很多东西都在我的demo中用到，[demo地址](https://github.com/MrZhang123/Vue_project/tree/master/vue_spa_demo)
<!--more-->
## 1.Vuejs组件
vuejs构建组件使用
```js
Vue.component('componentName',{ /*component*/ })；
```
这里注意一点，组件要先注册再使用，也就是说：
```js
Vue.component('mine',{
           template:'#mineTpl',
           props:['name','title','city','content']
        });

 var v=new Vue({
      el:'#vueInstance',
      data:{
          name:'zhang',
          title:'this is title',
         city:'Beijing',
         content:'these are some desc about Blog'
     }
});
```
如果反过来会报错，因为反过来代表先使用了组件的，但是组件却没注册。
> webpack报错后，使用`webpack --display-error-details`可以排错

## 2.指令keep-alive
在看demo的时候看到在vue-router写着`keep-alive`，`keep-alive`的含义：
如果把切换出去的组件保留在内存中，可以保留它的状态或避免重新渲染。为此可以添加一个keep-alive指令
```js
<component :is='curremtView' keep-alive></component>
```
## 3.如何让css只在当前组件中起作用
在每一个vue组件中都可以定义各自的css，js，如果希望组件内写的css只对当前组件起作用，只需要在`style`中写入`scoped`，即：
```css
<style scoped></style>
```
## 4.vuejs循环插入图片
在写循环的时候，写入如下代码：
```html
<div class="bio-slide" v-for="item in items">   
    <img src="{{item.image}}">
</div>
```
此时在控制台会出现警告
`[Vue Warn]: src="{{item.image}}": interpolation in "src" attribute will cause a 404 request. Use v-bind:src instead.`这里意思是在“src”属性插值将导致404请求。使用v-bind：src代替。
所以替换成如下：
```html
<div class="bio-slide" v-for="item in items">   
    <img v-bind:src="item.image">
</div>
```
这里需要主要，v-bind在写的时候不能再用双花括号，根据官方的说法：
```html
<a v-bind:href="url"></a>
```
> 这里` href `是参数，它告诉 `v-bind `指令将元素的 `href `特性跟表达式 url 的值绑定。可能你已注意到可以用特性插值` href="{{url}}"` 获得同样的结果：这样没错，并且实际上在内部特性插值会转为` v-bind` 绑定。

## 5.绑定value到Vue实例的一个动态属性上
对于单选按钮，勾选框及选择框选项，`v-model`绑定的value通常是静态字符串（对于勾选框是逻辑值）：
```html
<!-- `toggle` 为 true 或 false -->
<input type="checkbox" v-model="toggle">
```
但是有时候想绑定value到vue实例的一个动态属性上，这时可以用`v-bind`实现，并且这个属性的值可以不是字符串。例如绑定Checkbox的value到vue实例的一个动态属性：
```html
<input 
	type="checkbox"
	v-model="toggle"
	v-bind:true-value="a"
	v-bind:false-value="b">
<p>{{toggle}}</p>
```
这里绑定后，并不是说就可以点击后由`true`,`false`的切换变为`a`,`b`的切换，因为这里定义的动态a，b是scope上的a,b，并不能直接显示出来，此时
```js
//当选中时
vm.toggle === vm.a
//当没选中时
vm.toggle === vm.b
```
所以此时需要在data中定义a,b，即：
```js
new Vue({
    el:'...',
    data:{
        a:'a',
        b:'b'    
    }
});
```
## 6.片段实例
下面几种情况会让实例变成一个片断实例：
1. 模板包含多个顶级元素。
2. 模板只包含普通文本。
3. 模板只包含其它组件（其它组件可能是一个片段实例）。
4. 模板只包含一个元素指令，如`<partial>` 或` vue-router` 的 `<router-view>`。
5. 模板根节点有一个流程控制指令，如` v-if `或` v-for`。

这些情况让实例有未知数量的顶级元素，它将把它的 DOM 内容当作片断。片断实例仍然会正确地渲染内容。不过，它**没有**一个根节点，它的`$el` 指向一个锚节点，即一个空的文本节点（在开发模式下是一个注释节点）。
但是更重要的是，**组件元素上的非流程控制指令，非 prop 特性和过渡将被忽略**，因为没有根元素供绑定：
```html
<!-- 不可以，因为没有根元素 -->
<example v-show="ok" transition="fade"></example>
 
<!-- props 可以 -->
<example :prop="someData"></example>
 
<!-- 流程控制可以，但是不能有过渡 -->
<example v-if="ok"></example>
```
**片段实例**也有用处，但是通常情况下组件有一个根节点比较好，它会保证组件元素上的指令和特性能正确的转换，同时性能也稍微好些。

## 7.路由嵌套
**路由嵌套会将其他组件渲染到该组件内，而不是进行整个页面跳转**`router-view`本身就是将组件渲染到该位置，想要进行页面跳转，就要将页面渲染到根组件，在起始配置路由时候写到：
```js
var App = Vue.extend({ root });
router.start(App,'#app');
```
这里首先将根组件注册进来，用于将路由中配置好的各个页面渲染出来，然后将根组件挂载到与#app匹配的元素上。

## 8.实现多个根据不同条件显示不同文字的方法
`v-if`,`v-else`可以实现条件选择，但是如果是多个连续的条件选择，则需要用到计算属性`computed`。例如实现当输入框中什么都没写的时候显示字符串‘empty’,否则显示输入框中的内容，代码如下：
```html
<div id="test">
  <input type="text" v-model="inputValue">
  <h1>{{changeVaule}}</h1>
</div>
```
```js
new Vue({
  el:'#test',
  data:{
    changeVaule:'123'
  },
  computed :{
    changeVaule:function(){
      if(this.inputValue!==''){
        return this.inputValue;
      }else{
        return 'empty';
      }
    }
  }
});
```

## 9.Vuejs在变化检测问题
#### 1.检测数组
由于javascript的限制，vuejs不能检测到下面数组的变化：
1. 直接索引设置元素，如`vm.item[0]={}`;
2. 修改数据的长度，如`vm.item.length`。

为了解决问题1，Vuejs扩展了观察数组，为它添加一个`$set()`方法：
```js
// 与 `example1.items[0] = ...` 相同，但是能触发视图更新
example1.items.$set(0, { childMsg: 'Changed!'})
```
问题2，需要一个空数组替换`items`。

除了`$set()`，vuejs也为观察数组添加了`$remove()`方法，用于从目标数组中查找并删除元素，在内部调用了`splice()`。因此，不必：
```js
var index = this.items.indexOf(item)
if (index !== -1) {
  this.items.splice(index, 1)
}
```
只需：
```js
this.items.$remove(item);
```

#### 2.检测对象
受ES5的显示，Vuejs**不能检测到对象属性的添加或删除**。因为Vuejs在初始化时候将属性转化为`getter/setter`，所以属性必须在`data`对象才能让Vuejs转换它，才能让它是响应的，例如：
```js
var data = { a: 1 }
var vm = new Vue({
  data: data
})
// `vm.a` 和 `data.a` 现在是响应的
 
vm.b = 2
// `vm.b` 不是响应的
 
data.b = 2
// `data.b` 不是响应的
```
不过，有办法在实例创建之后添加属性并且让它是响应的。对于Vue实例，可以使用`$set(key,value)`实例方法：
```js
vm.$set('b', 2)
// `vm.b` 和 `data.b` 现在是响应的
```
对于普通数据对象，可以使用全局方法`Vue.set(object, key, value)`:
```js
Vue.set(data, 'c', 3)
// `vm.c` 和 `data.c` 现在是响应的
```
有时你想向已有对象上添加一些属性，例如使用 `Object.assign()` 或 `_.extend()` 添加属性。但是，添加到对象上的新属性不会触发更新。这时可以创建一个新的对象，包含原对象的属性和新的属性：
```js
// 不使用 `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```
## 10.关于vuejs页面闪烁`{{message}}`
在vuejs指令中有`v-cloak`，这个指令保持在元素上直到关联实例结束编译。和CSS规则如`[v-cloak]{display:none}`一起用时，这个指令可以隐藏未编译的Mustache标签直到实例准备完毕。用法如下：
```css
[v-cloak]{
    display:none;
}
```
```html
<div v-cloak>{{message}}</div>
```
这样`<div>`不会显示，直到编译结束
## 11.关于在`v-for`循环时候`v-model`的使用
有时候需要循环生成`input`，用`v-model`绑定后，利用vuejs操作它，此时我们可以在`v-model`中写一个数组`selected[$index]`，这样就可以给不同的input绑定不同的`v-model`，从而分别操作他们。这个我在demo中的dataBind.vue中用到。
## 12.vuejs中过渡动画
在vuejs中，css定义动画：
```css
  .zoom-transition{
        width:60%;
        height:auto;
        position: absolute;
        left:50%;
        top:50%;
        transform: translate(-50%,-50%);
        -webkit-transition: all .3s ease;
        transition: all .3s ease;
    }
    .zoom-enter, .zoom-leave{
        width:150px;
        height:auto;
        position: absolute;
        left:20px;
        top:20px;
        transform: translate(0,0);
    }
```
其中动画在定的时候要注意上下对应，上面有什么，下面有什么，都要变化的，如果有不变化的，应该抽离出去，作为公共css样式，在上面的css中，如果我只写` transform: translate(-50%,-50%);`而不写下面的`transform: translate(0,0);`则会导致上面的`transform: translate(-50%,-50%);`被添加到下面，认为这个是不变的。
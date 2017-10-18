---
title: Javascript中Array方法的总结
date: 2016-08-03 00:01:05
tags: Javascript
comments: true
categories: "Javascript"
---
> 在ECMAScript中最常用的类型之一就是Array类型，Array类型的方法也有很多，所以在这篇文章中，梳理一下Array类型的方法。

## 新建数组

新建数组的方法有三种：

```js
/*方法一*/
var a = new Array(1,2,3);
/*方法二*/
var b = [1,2,3];
/*方法三（ES6新增）*/
var c = Array.of(1,2,3);
```
`Array.of()`是ES6中新增的将一组值转换为数组的方法，该方法的出现时为了弥补构造函数`Array()`因为参数不同导致的不同行为。

```js
Array()         //[]
Array(3)        //[ , , ]
Array(1,2,3)    //[1,2,3]
```
从上面可以看出，只有在参数个数不少于2时候，才会返回新的数组。
<!--more-->
## 数组的检测
对于一个网页或者一个全局作用域而言，使用`instanceof`操作符检测，通过返回的`boolean`值可以得出是否为数组，但是这样检测的问题在如果网页中包含两个以上不同的全局作用域，就会从在两个以上不同版本的Array构造函数，如果从一个框架向另一个框架传入一个数组，那么传入的数组与第二个框架中原声创建的数组分别有不同的构造函数。
在ES5中引入的`Array.isArray()`解决了这个问题，但如果在不支持ES5的浏览器中检测数组，则需要些兼容性方法，所以检测数组的方法如下：
```js
function checkArray(arr) {
    if(typeof Array.isArray){
        return Array.isArray(arr);
    }else{
        return Object.prototype.toString.call(arr)==='[object Array]';
    }
}
```
## 数组中的方法：

### 更改原数组

#### 添加项

push():接收任意数量的参数，逐个将其**添加**至数组**末尾**，返回修改后的**数组的长度**
unshift():在数组的**前端添加**任意个项并返回新数组的长度

#### 移除项

pop():从数组**末尾移除最后**一项，返回移除的项
shift():**移除**数组中的**第一项**并返回该项

#### 排序

reverse():反转数组项的顺序

```js
var values = [1,2,3,4,5];
values.reverse();
console.log(values); // =>5,4,3,2,1
```
sort():按照升序排列数组项，但是它在实现排序时会调用每个数组项的`toString()`放法，去比较字符串，所以会出现如下情况

```js
var values = [0,1,5,10,15];
values.sort();
console.log(values); // => 0,1,10,15,5
```

为了在使用`sort()`方法时返回正确的排序，我们需要给`sort()`传入一个比较函数，该比较函数传入两个参数，如果第一个参数应该位于第二个参数之前则返回一个负数，如果两个参数相等返回0，如果第一个参数应该位于第二个参数之后则返回一个正数。

```js
/*升序
降序则更改返回值即可*/
function compare(value1,value2){
    if(value1 < value2){
        return -1;
    }else if(value1 > value2){
        return 1;
    }else {
        return 0;
    }
}
var values = [0,1,5,10,15];
values.sort(compare);
console.log(values);
```
对于数值类型或者其`valueOf()`方法会返回数值类型的对象类型，可以使用一个简单的比较函数

```js
function compare(value1,value2){
    return value2 - value1;
}
```

#### 截取
slice():接受一个或两个参数，要返回的起始位置到结束位置**但不包括结束位置**项，如果只写一个参数则截取数组到最后。**可以接收负数作为参数**
splice():做多可以接收三个参数，分别为*起始位置*，*要删除的项目数*，*要插入的任意数量的项*，同个这三个参数是否传入可以实现**删除**，**插入**，**替换**

```js
var colors =['red','green','blue'];
var removed =colors.splice(0,1);    //删除第一项
console.log(colors);                //green,blue
console.log(removed);               //redm,返回的数组中只包含一项

removed = colors.splice(1,0,'yellow','orange'); //从位置1开始插入两项
console.log(colors);                //green,yellow,orange,blue
console.log(removed);               //返回的是一个空数组

removed = color.splice(1,1,'red','purple');  //插入两项，删除一项
console.log(colors);                //green,yellow,purple,orange,blue
console.log(removed);               //返回yellow
```

#### copyWithin()

参数：
- target(必需)：从该位置开始替换数据
- start (可选)：从该位置开始读取数据，默认为0。如果为负值，表示倒数
- end   (可选)：到**该位置前**停止读取数据，默认等于数组长度。如果为负值表示倒数

在当前数组内部将指定位置的成员复制到其他位置，会覆盖原来的成员。修改原来的数组形成新的数组

```js
var a = [1,2,3];
var b = a.copyWithin(0);    // =>[1,2,3]
var c = a.copyWithin(0,1);  // =>[2,3,3]
var d = a.copyWithin(0,1,2);// =>[2,2,3]
```
上面例子可以看出，虽然`copyWithin`的后两个参数是可选的，但是需要写第二个参数，否则返回的只是原数组本身。

### 不更改原素组，生成新数组

#### 操作方法

concat():这个方法会先创建当前数组的一个副本，然后将接收到的参数添加到这个副本的末尾并返回副本。

```js
var a = [1,2,3];
var b = a.concat('a','b',['c','d','e']);
console.log(a); // =>1,2,3
console.log(b);
```

#### 寻找

##### 1.`indexOf()`、`lastIndexOf()`与`includes()`

`indexOf()`与`lastIndexOf()`用于查找数组中是否有该方法，如果有则返回该元素的位置，否则返回`-1`。
但是这个方法有两个缺点：

1. 不够语义化
2. 它内部使用严格等于运算符`===`，导致了对`NaN`的误判。所以ES7新增`includes()`去克服这些缺点。

**ES7新增**
ES7中新增`includes()`方法，用于查找数组总是否包含某个元素，返回布尔值，接受两个参数*要查找的元素*和*查找的起始位置*。

##### 2.`find()`和`findIndex()`

参数：*一个回掉函数*
回调的参数：*当前值*、*当前位置*、*原数组*

`find()`方法用于找出**第一个符合**条件的数组成员。
`findIndex()`方法返回**第一个符合**条件的数组成员的**位置**，如果所有成员都不符合，则返回-1。

```js
var a = [1,4,-5,10];
a.find((n)=> n<0);  // -5

var b = [1,5,10,15];
b.findIndex(function(value,index,arr){
    return value > 9;
});//=>2
```


#### 迭代方法

ES5为数组定义了五个迭代方法
每个方法都接收两个参数：*要在每一项上运行的函数*和*（可选的）运行该函数的作用域对象*----影响`this`的值。
传入这些方法中的函数会接收三个参数：*数组项的值*、*该项在数组中的位置*和*数组对象本身*。

- every():对数组中的每一项运行给定的函数，如果该函数对**每一项**都返回`true`,则返回`true`
- some():对数组中每一项运行给定的函数，如果该函数对**任一项**返回`true`，则返回`true`
- filter():对数组中每一项运行给定的函数，返回该函数会返回`true`的项组成的**数组**
- forEach():对数组中的每一项运行给定的函数。没有返回值
- map():对数组中的每一项运行给定的函数，返回每次调用的结果组成的**数组**

#### 归并方法

##### reduce()

该方法可以传递两个参数：*化简函数*，*传递给函数的初始值（可选）*。
化简函数的参数：*到目前为止的化简操作累积的结果*，*数组元素*，*元素的索引*，*数组本身*。
这个方法，可以用于求数组元素的和、积、最大值。

```js
var a = [1,2,3,4,5]
/*求和*/
var sum = a.reduce((x,y)=>x+y,0);
/*求积*/
var product = a.reduce((x,y)=>x*y,1);
/*求最大值*/
var max = a.reduce((x,y)=>(x>y)?x:y);
```
这个方法的简单用法就是这样，在《javascript高级程序设计》（第三版）中只是介绍了这个用法，但是在《javascript权威指南》（第六版）中提到了`reduce`的**高级用法。**
例1：求任意数目对象的“并集”

```js
/*
返回一个新对象，这个对象同时拥有o和p的属性
如果o和p中有重名属性，使用p中属性
*/
function union(o,p){
    return extend(extend({},o),p);
}
var objects = [{x:1},{y:2},{z:3}];
var merged = objects.reduce(union); // =>{x:1,y:2,z:3}
```

例2：统计字符串中每个字符出现的重复次数

```js
var arr = 'abcdabcdadbc';
var info = arr.split('').reduce((p,k) => (p[k]++ || (p[k] = 1), p), {});
console.log(info); //=> Object {a: 3, b: 3, c: 3, d: 3}
```

这两个例子，尤其是第二个例子可以看出，`reduce()`并不单单只是用于数学计算，在第二个例子中可以明显看出在`reduce()`第二个参数传入一个空对象，此时它最终返回的就是一个对象。由于本身传入的初始值是对象，所以返回对象。如果传入一个空数组，则返回数组。所以可以看出，最终`reduce()`函数返回什么，取决于第二个参数的形式。

#### join()

`Array.join()`方法将数组中的所有元素都转化为字符串并连接起来，返回最后生成的字符串。可以指定一个可选的字符串在生成的字符串中分隔数组的各个元素，如不指定，默认用逗号隔开。

#### fill()

参数：*填充项*、*填充的起始位置*、*填充的结束位置*
`fill()`方法用于使用给定的值填充数组。

```js
new Array(3).fill(7); //=>[7,7,7]
```

## 转换为数组的方法(ES6新增)

#### Array.from();

该方法接收两个参数*要转换的非数组对象*,*对每个元素进行处理的方法（可选）*

在js中，有很多类数组对象（array-like object）和可遍历（iterable）对象（包括ES6新增的数据结构Set和Map），常见的类数组对象包括`document.querySelectorAll()`取到的NodeList，以及函数内部的arguments对象。它们都可以通过`Array.from()`转换为真正的数组，从而使用数组的方法。事实上只要对象具有`length`属性，就可以通过`Array.from()`转换为真正的数组。

```js
var a = {
    0:'li',
    1:'li',
    2:'li',
    length:3
};
console.log(Array.from(a)); // => ['li','li','li'];
```
```js
Array.from([1,2,3],(x)=>x*x); // =>1,4,9
```

#### 扩展运算符（...）

```js
//arguments对象
function foo(){
    var args = [...arguments];
}
//nodelist
[...document.querySelectorAll('div')];
```

---
title: 理解CSS命名规范--BEM
date: 2017-04-05 20:57:48
tags: CSS
comments: true
categories: "CSS"
---

>最近在写博客主题的时候发现一个很严重的问题，由于css的命名并不是很规范，导致自己在后期修改的时候很是头疼，有些样式需要在浏览器中打开开发者工具去找，很是无奈。所以决定在写完主题之后学习下CSS命名规范中大名鼎鼎的BEM命名规范。

<!--more-->

### 什么是BEM

BEM其实是块（block）、元素（element）、修饰符（modifier）的缩写，利用不同的区块，功能以及样式来给元素命名。这三个部分使用`__`与`--`连接（这里用两个而不是一个是为了留下用于块儿的命名）。命名约定的模式如下：

```css
.block{}
.block__element{}
.block--modifier{}
```

- `block` 代表了更高级别的抽象或组件
- `block__element` 代表 `block` 的后代，用于形成一个完整的 `block` 的整体
- `block--modifier`代表 `block` 的不同状态或不同版本


![bem](/img/bem.jpg)


上图很直观的反映出BEM的含义，block代表一个组，而element代表组里面的成员，而modifier，虽然在上图没有画出，但是可以知道这个则是用于描述每一个元素的具体的属性。可以看出范围在一步步缩小，使命名更加具体。

### BEM的优势

BEM的关键是光凭class名字就可以让其他开发者知道某个标记用来做什么的，明白各个模块之间的关系，例如如下的命名：

```css
.nav{}
.nav__item{}
.nav--blue{}
.nav--item__hand{}
```

这几个class名很明显能看出各个块儿的作用，顶级快儿是`nav`，它有一些元素比如`item`，`item`又有一些属性，例如`blue`，但是如果写成常规的css就会很难看得出它们的关系：

```css
.nav{}
.item{}
.blue{}
```

虽然看每一个class名知道它们代表什么，但是却看不出它们之间的关系，这样对比，很明显能看出BEM命名的优势。

再看一个更具体的例子，如果写一个搜索模块，按照常规，我们会写出如下代码：

```html
<form class="site-search  full">
  <input type="text" class="field">
  <input type="Submit" value ="Search" class="button">
</form>	
```

但是如果时用BEM规范去写，代码如下：

```html
<form class="site-search  site-search--full">
  <input type="text" class="site-search__field">
  <input type="Submit" value ="Search" class="site-search__button">
</form>	
```

对比一下不难发现使用BEM可以使我们的代码可读性更高。

### BEM与SCSS

现在的开发很多时候都会用到SCSS，那么如果使用SCSS的嵌套写BEM规范呢，在SCSS中可以使用`@at-root`：

```scss
.block {
  @at-root #{&}__element {
  }
  @at-root #{&}--modifier {
  }
}
```

```css
/*生成的css*/
.block {
}
.block__element {
}
.block--modifier {
}
```

非常的方便。

### 最后

在自己的博客主题项目中，因为命名的随意导致后来再看样式的时候都需要通过开发者工具去看具体这部分是做什么的，非常难受，所以想到应该用一套规范约束一下命名，而BEM最为一个非常有用，强大的命名规范可以让我们的代码更容易阅读和理解，也更容易控制，虽然这种命名方式看起来有点儿奇怪，但是却非常有用，非常值得学习。
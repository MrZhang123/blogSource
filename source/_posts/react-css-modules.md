---
title: CSS变量污染解决及CSS Modules在React中的应用
date: 2018-04-13 14:32:17
tags: React
comments: true
categories: "React"
---
# CSS变量污染解决及CSS Modules在React中的应用

由于CSS是全局的，所以在写组件的时候，经常会遇到CSS命名重复导致样式覆盖（冲突），所以我们在写CSS的时候一般会这么处理

* 写复杂的class名，降低冲突的概率
* 给组件最外层元素添加一个class，限制范围

<!--more-->

但是这样做也不一定可以保证不会冲突，而且还导致class名字太复杂，嵌套太深，可维护性差，那么是否可以将CSS也像JS那样，实现模块化呢？答案是肯定的。CSS模块化方案很多，但是主要的就三类：

#### 命名约定

比较常用的有[BEM](http://mrzhang123.github.io/2017/04/05/BEM/)，SMACSS和OOCSS，但是存在以下问题：

* JS CSS之间依旧没有打通变量和选择器
* 命名太复杂

#### CSS in JS

直接在JS中写CSS并内联样式，例如aphrodite，babel-plugin-css-in-js等（[点击查看所有CSS in JS的解决方案](http://michelebertoli.github.io/css-in-js/)）但是存在以下问题：

* 样式代码可能会重复出现
* 写法上已经和传统的CSS不再相似（例如[aphrodite](https://github.com/Khan/aphrodite)，写法类似于React Native中样式的写法）
* 不能利用成熟的CSS预处理器（或后处理器）

#### 使用JS来管理CSS模块

使用JS编译原生CSS文件，使其具有模块化，典型的就是[CSS Modules](https://github.com/css-modules/css-modules)。只要使用到webpack，就会使用到css-loader，在webpack中稍加配置即可使用

### 使用CSS Modules

配置css-loader启动css modules

```js
// webpack.config.js
  {
    test: /\.css$/,
    use: ExtractTextPlugin.extract({
      fallback: 'style-loader',
      use: {
        loader: 'css-loader',
        options:{
          modules: true,
          minimize: true,
          localIdentName: '[path][name]__[local]--[hash:base64:5]'
        }
      }
    })
  },
```

```css
// Button.css
.button{
  font-size: 10px;
}
```

```js
// Button.js
import styles from './Button.css'
console.log(styles)
buttonElement.outerHTML = `<div class=${styles.button}>Button</div>`
```

console出来的styles如下：

```js
{
 button:"src-components-Button-index__button--1mmZb"
 large:"src-components-Button-index__large--2atzR"
 normall:"src-components-Button-index__normall--3prnh"
 small:"src-components-Button-index__small--34Wrr"
}
```

通过以上配置，css loader为我们生成如上class名字，其中`1mmZb`是按照`[hash:base64:5]`生成的，大大降低了命名冲突的概率。

通过这些简单的处理，CSS Modules 实现了以下几点：

* 所有样式都是局部作用域的，解决了全局污染问题
* class 名生成规则配置灵活，可以此来压缩 class 名
* 只需引用组件的 JS 就能搞定组件所有的 JS 和 CSS
* 依然是熟悉的CSS，学习成本低

### 在React中使用CSS Modules

直接在className处使用css中的class名即可

```js
import React, { Component } from 'react'
import { render } from 'react-dom'
import styles from './index.css'

class Button extends Component {
  constructor(props) {
    super(props)
  }
  render() {
    console.log(styles)
    let buttonClass = ''
    switch (size) {
      case 'large':
        buttonClass = styles.large
        break

      case 'small':
        buttonClass = styles.small
        break

      default:
        buttonClass = styles.normall
        break
    }
    return (
      <button className={buttonClass}>确定</button>
    )
  }
}

export default Button
```

注

1.使用CSS Modules时发现，它只支持单独的class名字，不能像我们写css的时候一级一级的写，例如：`.a .b .c`，在CSS Modules中就是一步到位`.c`。

2.CSS Modules提供了compose组合方法实现样式的复用，代码如下：

```css
.font {
 line-height: 12px;
 font-size: 12px;
}

.title-font {
 composes: font;
 font-size: 24px;
}
```

#### React CSS Modules

但是有一个问题，我们在写样式的时候，需要频繁使用 styles.xxx，如何能够方便的直接写入class名字呢？可以使用[React CSS Modules](https://github.com/gajus/react-css-modules)，它以高阶函数的形式生成className

```js
import React, { Component } from 'react'
import { render } from 'react-dom'
import CSSModules from 'react-css-modules'
import styles from './index.css'

class Button extends Component {
  render() {
    return (
      <button styleName='normall'}>确定</button>
    )
  }
}

export default CSSModules(Button, styles)
```

可以看到，react-css-modules是运行时的依赖，而且需要在运行时获取className，性能损耗比较大，可否把获取className前置到编译阶段？答案是可以的，可以使用[babel-plugin-react-css-modules](https://github.com/gajus/babel-plugin-react-css-modules)

#### babel-plugin-react-css-modules

babel-plugin-react-css-modules插件可以实现用styleName属性自动加载CSS模块，通过babel插件来进行语法树解析并最终生成className，写的时候，只需要将我们原来写的className替换成styleName即可。

```js
import React, { Component } from 'react'
import { render } from 'react-dom'
import styles from './index.css'

class Button extends Component {
  render() {
    return (
      <button styleName='normall'}>确定</button>
    )
  }
}

export default Button
```

具体配置可以查看[文档](https://github.com/gajus/babel-plugin-react-css-modules#babel-plugin-react-css-modules)

### 参考

https://github.com/webpack-contrib/css-loader 

https://juanha.github.io/2017/01/10/react-css-modules/ 

http://www.alloyteam.com/2017/03/getting-started-with-css-modules-and-react-in-practice/ 

https://github.com/camsong/blog/issues/5
---
title: Webpack中publicPath详解
date: 2018-05-12 12:26:43
tags: webpack
comments: true
categories: "webpack"
---
>最近自己在搭建一个基于webpack的react项目，遇到关于`output.publicPath`和webpack-dev-server中`publicPath`的问题，而官方文档对它们的描述也不是很清楚，所以自己研究了下并写下本文记录。

<!--more-->

## output

output选项指定webpack输出的位置，其中比较重要的也是经常用到的有`path`和`publicPath`

### output.path

* 默认值：`process.cwd()`

`output.path`只是指示输出的目录，对应一个**绝对路径**，例如在项目中通常会做如下配置：

```js
output: {
 path: path.resolve(__dirname, '../dist'),
}
```

### output.publicPath

* 默认值：空字符串

[官方文档中对publicPath的解释](https://doc.webpack-china.org/guides/public-path/)是

>webpack 提供一个非常有用的配置，该配置能帮助你为项目中的所有资源指定一个基础路径，它被称为公共路径(publicPath)。

而关于如何应用该路径并没有说清楚...

其实这里说的所有资源的基础路径是指项目中引用css，js，img等资源时候的一个基础路径，这个基础路径要配合具体资源中指定的路径使用，所以其实打包后资源的访问路径可以用如下公式表示：

```shell
静态资源最终访问路径 = output.publicPath + 资源loader或插件等配置路径
```

例如

```javascript
output.publicPath = '/dist/'

// image
options: {
  name: 'img/[name].[ext]?[hash]'
}

// 最终图片的访问路径为
output.publicPath + 'img/[name].[ext]?[hash]' = '/dist/img/[name].[ext]?[hash]'

// js output.filename
output: {
 filename: '[name].js'
}
// 最终js的访问路径为
output.publicPath + '[name].js' = '/dist/[name].js'

// extract-text-webpack-plugin css
new ExtractTextPlugin({
 filename: 'style.[chunkhash].css'
})
// 最终css的访问路径为
output.publicPath + 'style.[chunkhash].css' = '/dist/style.[chunkhash].css'
```

这个最终静态资源访问路径在使用html-webpack-plugin打包后得到的html中可以看到。所以`publicPath`设置成相对路径后，相对路径是相对于build之后的index.html的，例如，如果设置`publicPath: './dist/'`，则打包后js的引用路径为`./dist/main.js`，但是这里有一个问题，相对路径在访问本地时可以，但是如果将静态资源托管到CDN上则访问路径显然不能使用相对路径，但是如果将`publicPath`设置成`/`，则打包后访问路径为`localhost:8080/dist/main.js`，本地无法访问

所以这里需要在上线时候手动更改`publicPath`，感觉不是很方便，但是不知道该如何解决...

>一般情况下**publicPath应该以'/'结尾，而其他loader或插件的配置不要以'/'开头**

## webpack-dev-server中的publicPath

[点击查看官方文档中关于devServer.publicPath的介绍](https://doc.webpack-china.org/configuration/dev-server/#devserver-publicpath-)

在开发阶段，我们借用devServer启动一个开发服务器进行开发，这里也会配置一个`publicPath`，这里的`publicPath`路径下的打包文件可以在浏览器中访问。而静态资源仍然使用`output.publicPath`。

webpack-dev-server打包的内容是放在内存中的，这些打包后的资源对外的的根目录就是`publicPath`，换句话说，这里我们设置的是打包后资源存放的位置

例如：

```javascript
// 假设devServer的publicPath为
const publicPath = '/dist/'
// 则启动devServer后index.html的位置为
const htmlPath = `${pablicPath}index.html`
// 包的位置
cosnt mainJsPath = `${pablicPath}main.js`
```

以上可以直接通过`http://lcoalhost:8080/dist/main.js`访问到。

通过访问 `http://localhost:8080/webpack-dev-server` 可以得到devServer启动后的资源访问路径，如图所示，点击静态资源可以看到静态资源的访问路径为 http://localhost:8080${publicPath}index.html

![webpack-dev-server资源访问路径](/img/webpack/webpack-publicpath.png)


## html-webpack-plugin

这个插件用于将css和js添加到html模版中，其中`template`和`filename`会受到路径的影响，从源码中可以看出

### template

作用：用于定义模版文件的路径

源码：

```javascript
this.options.template = this.getFullTemplatePath(this.options.template, compiler.context);
```

因此`template`只有定义在webpack的`context`下才会被识别，**webpack context的默认值为`process.cwd()`，既运行 node 命令时所在的文件夹的绝对路径**

### filename

作用：输出的HTML文件名，默认为index.html，可以直接配置带有子目录

源码：

```javascript
this.options.filename = path.relative(compiler.options.output.path, filename);
```

**所以filename的路径是相对于`output.path`的，而在webpack-dev-server中，则是相对于webpack-dev-server配置的`publicPath`。**

如果webpack-dev-server的`publicPath`和`output.publicPath`不一致，在使用html-webpack-plugin可能会导致引用静态资源失败，因为在devServer中仍然以`output.publicPath`引用静态资源，和webpack-dev-server的提供的资源访问路径不一致，从而无法正常访问。

>有一种情况除外，就是`output.publicPath`是相对路径，这时候可以访问本地资源

**所以一般情况下都要保证devServer中的`publicPath`与`output.publicPath`保持一致。**

## 最后

关于webpack中的`path`就总结这么多，在研究关于webpack路径的过程中看查到的一些关于路径的零碎的知识如下：

### 斜杠`/`的含义

配置中`/`代表url根路径，例如`http://localhost:8080/dist/js/test.js`中的`http://localhost:8080/`

### devServer.publicPath & devServer.contentBase

* devServer.contentBase 告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。
* devServer.publicPath 将用于确定应该从哪里提供 bundle，并且此选项优先。

### node中的路径

* `__dirname`: 总是返回被执行的 js 所在文件夹的绝对路径
* `__filename`: 总是返回被执行的 js 的绝对路径
* `process.cwd()`: 总是返回运行 node 命令时所在的文件夹的绝对路径

## 参考

* [详解Webpack2的那些路径](http://www.qinshenxue.com/article/20170315092242.html)
* [webpack 公共路径(Public Path)](https://doc.webpack-china.org/guides/public-path/)
* [devServer.publicPath](https://doc.webpack-china.org/configuration/dev-server/#devserver-publicpath-)
* [浅析 NodeJs 的几种文件路径](https://github.com/imsobear/blog/issues/48)
* [项目中关于相对路径和绝对路径的问题](https://blog.csdn.net/m0_37721946/article/details/78340638)
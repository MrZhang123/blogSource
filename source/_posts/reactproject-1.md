---
title: 从零开始搭建React应用（一）——基础搭建
date: 2018-07-18 10:26:46
tags: React
comments: true
categories: "React"
---
项目链接：[https://github.com/MrZhang123/Web_Project_Build/tree/master/react-webpack](https://github.com/MrZhang123/Web_Project_Build/tree/master/react-webpack)

<!--more-->

## 核心

* React:16.3.2
* React-dom:16.3.2
* Webpack:4.6.0
* React-router-dom:4.2.2
* Redux:4.0.0
* React-hot-loader:4.1.2

## 目录结构

```sh
├── README.md
├── build
│   ├── webpack.base.config.js
│   ├── webpack.dev.config.js
│   └── webpack.production.config.js
├── package.json
├── postcss.config.js
├── src
│   ├── App.js
│   ├── components
│   │   └── Loading
│   │       └── index.js
│   ├── index.html
│   ├── main.js
│   ├── redux
│   │   ├── actions
│   │   │   └── demo.js
│   │   ├── middleware
│   │   │   └── index.js
│   │   ├── reducers
│   │   │   ├── demo.js
│   │   │   └── index.js
│   │   └── store
│   │       └── index.js
│   ├── static
│   │   └── css
│   │       └── normalize.css
│   ├── styles.scss
│   └── views
│       ├── Content
│       │   ├── About
│       │   │   └── index.js
│       │   ├── Home
│       │   │   └── index.js
│       │   ├── Topics
│       │   │   └── index.js
│       │   └── router.js
│       └── RouterLink
│           └── index.js
└── yarn.lock
```

## 基础搭建

首先搭建一个基础的React开发环境，没有redux和react-router

### 安装必须的包

```sh
# dependencies
yarn add react react-dom

# devDependencies
yarn add --dev webpack webpack-cli webpack-dev-server babel-core babel-loader babel-polyfill babel-preset-env babel-preset-react babel-preset-stage-0 cross-env css-loader file-loader jsx-loader style-loader url-loader
```

**说明**

1. webpack4的cli(command line interface)已经移动到webpack-cli了，如果要使用CLI,你需要安装webpack-cli，具体使用可以查看[webpack-cli的文档](https://doc.webpack-china.org/api/cli/)。
2. 由于Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign），所以如果想使用这些方法就必须使用babel-polyfill，为当前环境提供一个垫片。

### 配置webpack

在build文件夹下新建三个文件——webpack.base.config.js, webpack.dev.config.js, webpack.production.config.js，将dev和produciton里公共的部分放在base文件中。

```js
// webpack.base.config.js
const path = require('path')

module.exports = {
  entry: {
    main: [
      'babel-polyfill',
      path.resolve(__dirname, '../src/main.js')
    ]
  },
  output: {
    path: path.resolve(__dirname, '../dist'),
    publicPath: '/',
    filename: '[name].js',
    chunkFilename: 'chunk/[name].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(js|jsx)$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel-loader'
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: '[name].[ext]?[hash]'
        }
      }
    ]
  },
  resolve: {
    modules: ['node_modules'],
    extensions: ['.web.js', '.js', '.jsx', '.json']
  },
  mode: '' // webpack v4 add
}
```

由于是webpack的公共配置文件，所以这里mode设置空，之后在dev和prodctuon中再分别设置。

#### webpack4的mode

在webpack v4中新增一些默认配置，通过设置`mode`是`development`和`production`（默认）来启用在开发环境和生产环境的默认配置。

1.二者都有的配置

```js
//parent chunk中解决了的chunk会被删除
optimization.removeAvailableModules:true
//删除空的chunks
optimization.removeEmptyChunks:true
//合并重复的chunk
optimization.mergeDuplicateChunks:true
```

2.development的默认配置

```js
//调试
devtool:eval
//缓存模块, 避免在未更改时重建它们。
cache:true
//缓存已解决的依赖项, 避免重新解析它们。
module.unsafeCache:true
//在 bundle 中引入「所包含模块信息」的相关注释
output.pathinfo:true
//在可能的情况下确定每个模块的导出,被用于其他优化或代码生成。
optimization.providedExports:true
//找到chunk中共享的模块,取出来生成单独的chunk
optimization.splitChunks:true
//为 webpack 运行时代码创建单独的chunk
optimization.runtimeChunk:true
//编译错误时不写入到输出
optimization.noEmitOnErrors:true
//给模块有意义的名称代替ids
optimization.namedModules:true
//给模chunk有意义的名称代替ids
optimization.namedChunks:true
```

3.production的默认设置

```js
//性能相关配置
performance:{hints:"error"....}
//某些chunk的子chunk以一种方式被确定和标记,这些子chunks在加载更大的块时不必加载
optimization.flagIncludedChunks:true
//给经常使用的ids更短的值
optimization.occurrenceOrder:true
//确定每个模块下被使用的导出
optimization.usedExports:true
//识别package.json or rules sideEffects 标志
optimization.sideEffects:true
//尝试查找模块图中可以安全连接到单个模块中的段。- -
optimization.concatenateModules:true
//使用uglify-js压缩代码
optimization.minimize:true
```

#### 配置webpack.dev.config.js

```js
const webpack = require('webpack')
const config = require('./webpack.base.config.js')
const WebpackDevServer = require('webpack-dev-server')
const PORT = process.env.PORT || 8000 // 默认8000端口，可以通过package.json配置

config.entry.main = (config.entry.main || []).concat([
  `webpack-dev-server/client?http://localhost:${PORT}/`,
  'webpack/hot/dev-server'
])

config.plugins = (config.plugins || []).concat([
  new webpack.HotModuleReplacementPlugin()
])

config.mode = 'development'

const compiler = webpack(config)

const server = new WebpackDevServer(compiler, {
  hot: true, // 开启wbepack HMR
  quiet: true,
  historyApiFallback: true,
  filename: config.output.filename,
  publicPath: config.output.publicPath,
  stats: {
    colors: true
  }
})

server.listen(PORT, 'localhost', () => {
  console.log(`server started at localhost:${PORT}`)
})
```

关于`output.publicPath`和`devServer.publicPath`的区别可以参考我的文章：[Webpack中publicPath详解](https://juejin.im/post/5ae9ae5e518825672f19b094)

#### 配置webpack.production.config.js

```js
const path = require('path')
const webpack = require('webpack')
const config = require('./webpack.base.config')
const CleanWebpackPlugin = require('clean-webpack-plugin')

config.mode = 'production'

config.plugins = (config.plugins || []).concat([
	new CleanWebpackPlugin(['dist'], {
    root: path.resolve(__dirname, '../')
  }),
  new webpack.HashedModuleIdsPlugin()
])

module.exports = config
```

## 结语

至此，一个简单的React开发环境就搭建完成，但是在使用后会发现一些问题，比如webpack将js和css打包在一起，我们在使用webpack的HMR时候，只要保存，页面就会刷新，页面的状态会重置等问题以及加入常用的redux和react-router，下一篇——[从零开始搭建React应用（二）——React应用架构](https://juejin.im/post/5b4de4496fb9a04fc226a7af)将就这些问题作出解决。
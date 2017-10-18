---
title: Nodejs之npm&package.json
date: 2016-11-28 22:25:36
tags: Node
comments: true
categories: "Node"
---
>一直以来，作为前端开发，在公司都是先写好页面，然后再跟后端合作，将数据填入前端页面中，但是偶尔自己闲来无事，也会看一些框架什么的，然后利用框架做个单页面应用啊，app什么的，这时候页面的数据总是一些假数据，而关于数据请求的部分就没办法做（因为没有后台嘛）。所以我感觉是时候学习一下node了，这对于我以后要学的webpack，前端工程化等也有一定帮助。

<!--more-->

## 如何开始学习node

首先介绍几个我入门的教程：

七天学会nodejs：[https://nqdeng.github.io/7-days-nodejs/](https://nqdeng.github.io/7-days-nodejs/)

菜鸟教程-nodejs教程：[http://www.runoob.com/nodejs/nodejs-tutorial.html](http://www.runoob.com/nodejs/nodejs-tutorial.html)

这两个教程相对比较简单，所以看完这两个教程后，对nodejs就会也只会有一个大致的了解

然后可以看看止水大神的node教程（<font color='red'>需梯子</font>还在更新中...）

node.js高级编程：[https://www.youtube.com/watch?v=5YpYvrvVJ6Y&list=PLsdWTv8SAAr7_ufM68jgykoOc5WvK97kb](https://www.youtube.com/watch?v=5YpYvrvVJ6Y&list=PLsdWTv8SAAr7_ufM68jgykoOc5WvK97kb)

当初我问很多人如何开始学习nodejs，很多人推荐朴灵的《深入浅出Node.js》，不过说实话，这本书并不适合入门看，难度还是有的，因为这本书会讲到node的很多原理，比较细，所以个人觉得可以配合着看。因为目前我自己也在摸索着学习，所以关于学习node先说到这里。

作为前端，因为经常用到gulp，webpack等工具，所以我们最常见到的是npm和package.json，所以先总结一下它们俩。

## npm

#### 初始化

```shell
$ npm init
or
$ npm init --y 
```
在做前端开发的时候，我们经常会用到构建工具，例如gulp，webpack等，为了让别人也可以参与进来，我们需要告诉别人项目有些什么依赖包，然后让别人也安装同样的依赖包，而`npm init`产生的package.json就是用来记录我们项目中的依赖的，同样的，在做node开发的时候，也会用刀依赖包，同样需要package.json记录。

在终端输入`npm init`会询问package.json的各种信息，从而确认。如果**全部使用默认值**，可以直接在终端输入`npm init --y`快速生成package.json。


#### 安装依赖包

```shell
$ npm install <package name> <package name> ...

$ npm install <package name> -g

$ npm install <package name> --save

$ npm install <package name> --save-dev

$ npm install <pacakage name>  --O //--save-optional  -B: --save-bundle  -E: --save-exact
```

`npm install <package name> -g` 表示全局安装，需要注意的是全局模式并不是将一个模块安装包安装为一个全局包的意思，它并不意味着可以从任何地方通过`require()`来引用，`-g`的含义是将一个包安装为全局可用的可执行命令。这意味着，所有通过`-g`安装的包都可以在终端以命令方式运行，例如gulp，webpack等。

`--save`与`--save-dev`的区别在于前者是**生产环境中**项目运行需要的依赖，安装后被记录在package.json中的dependencies关键字下；而后者是**开发**时候需要的依赖，安装后被记录在devDependencies关键字下。

同样`--O/B/E`分别会被记录到对应的关键字下。

#### 更新依赖包

```shell
$ npm update

$ npm update  -g

$ npm outdated

$ npm outdated -g
```

在项目目录下运行`npm update`可以升级项目中所用依赖到最新版本，而`npm update -g`则可以升级全局安装的依赖包到最新版。

`npm outdated`用于**检查**模块是否过时并列出。

#### 卸载依赖

```shell
$ npm uninstall <package name> <package name> ...

$ npm uninstall <package name> -g

$ npm uninstall <package name> --save

$ npm uninstall <package name> --save-dev

```
使用`npm uninstall`可以卸载依赖，但是卸载后，在package.json中的纪录并不会被删除，要想在卸载依赖的同时删除在package.json中的纪录，需要在卸载的时候使用安装时的所有的选项，例如，如果安装时候使用了`npm install <package name> --save`则卸载的时候，同样使用`npm uninstall <pacakage name> --save`，而如果使用了`--save-dev`，卸载时候也需要加相同的选项。

#### 使用自定义npm命令

在package.json中，有一个`scripts`关键字，只需要在该关键字内写入自定义命令以及对应执行的实际命令即可。

```json
"scripts":{
    "test": "nonde ./test.js",
    "dev": "gulp --gulpfile gulpfile-dev.js",
    "build": "gulp --gulpfile gulpfile-build.js"
}
```

上面的配置中，只要我们在终端运行`npm dev`就是运行了`gulp --gulpfile gulpfile-dev.js`，这样就省去了我们在终端输入很长的一段命令，非常方便。

#### 其他

`npm view <pacakage name>`可以查看包的package.json文件，如果只是看包的某个特性，在后面加上相应的key即可，例如`npm v zepto version`就是查看当前安装的zepto的版本，v是view的简写。

`npm ls`可以分析出当前当前项目下能够通过模块路径找到的所有包，并生成依赖树。

`npm doc <package name>`可以打开该依赖包的官网，其实就是打开了package.json中的homepage。

## package.json文件

在运行`npm init`后会生成package.json文件，该文件用于记录项目中所用到的依赖以及项目的配置信息（比如名称、版本、许可证等）。`npm install`命令根据这个配置文件自动下载项目运行和开发所需要的依赖。

一个比较完整的package.json文件如下：

```json
{
	"name": "project",
	"version": "1.0.0",
	"author": "张三",
	"description": "第一个node.js程序",
	"keywords":["node.js","javascript"],
	"repository": {
		"type": "git",
		"url": "https://path/to/url"
	},
	"license":"MIT",
	"engines": {"node": "0.10.x"},
	"bugs":{"url":"http://path/to/bug","email":"bug@example.com"},
	"contributors":[{"name":"李四","email":"lisi@example.com"}],
	"scripts": {
		"start": "node index.js"
	},
	"dependencies": {
		"express": "latest",
		"mongoose": "~3.8.3"
	},
	"devDependencies": {
		"grunt": "~0.4.1",
		"grunt-contrib-concat": "~0.3.0"
	}
}
```

在package.json中一些关键字的含义：

1.name：包名

2.version：版本号

3.description：包的描述

4.homepage：包的官网url

5.autor：包的作者名字

6.contributors：包的其他贡献者

7.dependencies：依赖包的列表，使用`npm install`可以安装依赖包到node_medule目录下

8.repository：包代码存放的地方，可以是git或者svn

9.keywords：关键字

10.scripts：脚本说明对象。它主要被包管理器用来安装、编译、测试和卸载包，示例如下：

```json

"scripts":{

    “install”:"install.js",

    "test":"test.js"

}

```

11.main：模块引入方法require()在引入包时，会优先检查这个字段，并将其作为包中其余模块的入口，如果该字段不存在，则node会检查目录下的index.js，index.node，index.json作为默认入口。

12.devDependencies：一些模块只在开发时需要依赖，配置这个属性，可以提示包的后续开发者安装依赖包

以上就是关于node中npm和package.json的总结。最后在写本文的时候发现阮一峰老师的一个关于js的教程[JavaScript 标准参考教程](http://javascript.ruanyifeng.com/)，应该是阮一峰老师最新的关于js的教程，里面有关于node的讲解，可以看看。
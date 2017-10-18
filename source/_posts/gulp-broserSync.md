---
title: Gulp+BroserSync实现浏览器自动刷新
date: 2016-05-16 22:10:39
tags: gulp
comments: true
categories: "前端工具"
---
&emsp;&emsp;在写前端代码的时候，我们为了看效果，需要一直按F5进行刷新，这样做很繁琐而且非常浪费时间，在网上搜过后发现很多关于自动刷新的办法，这里我介绍的是基于gulp和broserSync实现浏览器的自动刷新，即只要编辑器保存，浏览器就会自动刷新。
&emsp;&emsp;那么gulp，broserSync是什么呢？
### Gulp
&emsp;&emsp;Gulp是一个前端自动化工具，基于nodejs，和grunt差不多，但是比grunt语法更加简单，语法更加自然。在gulp的插件中，我们可以找到自动刷新，压缩图片/代码/等等各类工具，方便我们的使用，并且gulp的任务是流（pipe），即一个任务完成后，紧接的另一个任务开始进行。gulp的使用如下：
#### 1.安装gulp
1. 全局安装gulp：
```js
$ npm install --global gulp
```
2. 作为项目的开发依赖（devDependencies）安装：
```js
$ npm install --save-dev gulp
```
3. 生成package.json
```js
$ npm init
```
<!--more-->
<font color=red>注意：</font>
&emsp;&emsp;这里官网只是写让这么安装，但是需要注意，两个安装不是第一步第二步，而是第一种方式或者第二种方式，两种方式任选其一。第一种安装方式是全局安装，即安装后，整个电脑无论哪个项目都可以使用。对应的，第二个表示安装到对应项目中，即如果在window的cmd命令行进入对应项目A的目录，则运行npm install --save-dev gulp后，安装的gulp只能在该文件夹下使用，其他文件夹下的其他文件都无法使用。
&emsp;&emsp;其实不只是gulp，gulp的插件也是这样，要么全局安装，要么安装在项目中。一般情况下，gulp会全局安装，但是由于每个项目用到的gulp插件不同，所以可能需要局部安装gulp插件。
&emsp;&emsp;安装完成gulp之后，会在安装gulp的文件夹下生成node_modules文件夹。此时，在与该文件夹同层创建gulpfile.js这就是用于配置gulp插件的文件。

#### 2.恢复gulp
&emsp;&emsp;随着我们gulp插件的改变，package.json会自动变化，同时我们的配置文件gulpfile.js也会对应变化（自己手动配置）。他们可以把我们安装的gulp给备份，如果我们在电脑A中安装完我们的工具，然后换了一台电脑B，我们只需要把电脑A中package.json与gulpfile.js复制到B电脑，然后在安装完gulp后，运行
```js
$ npm install
```
gulp就会自动把我们在package.json中的所有gulp插件全部安装回来，非常方便。
### broserSync
&emsp;&emsp;broserSync（以下简称bs）是一款非常优秀的自动刷新工具，本身可以独立使用，也可以配合gulp或者grunt一起使用，非常不错的一款插件。
&emsp;&emsp;当你改变代码的时候，BrowserSync会重新加载页面，或者如果是css文件，会直接添加进css中，页面并不需要再次刷新。这项功能在网站是禁止刷新的时候是很有用的。假设你正在开发单页应用的第4页，刷新页面就会导致你回到开始页。BrowserSync会直接将需要修改的地方添加进CSS，就不用再点击回退。同时，BrowserSync也可以在不同浏览器之间同步点击翻页、表单操作、滚动位置。你可以在电脑和iPhone上打开不同的浏览器然后进行操作。所有设备上的链接将会随之变化，当你向下滚动页面时，所有设备上页面都会向下滚动（通常还很流畅！）。当你在表单中输入文本时，每个窗口都会有输入。当你不想要这种行为时，也可以把这个功能关闭。
&emsp;&emsp;实际上bs对于gulp并不算是一种插件，因为bs并不像一个插件一样操作文件。然而，npm上的bs模块能在gulp上被直接调用。
#### 1.安装broserSync
进入需要使用bs的目录，运行：
```js
$ npm install --save-dev broser-sync
```
#### 2.配置broserSync
bs本身可以打开本地静态服务器，也可以代理像wamp这样的服务器。
##### 2.1 代理其他服务器
打开我们新建的gulpfile.js，配置如下：
```js
/*browserSync*/
const gulp        = require('gulp');
const browserSync = require('browser-sync').create();
gulp.task("watch",function(){
    browserSync.init({
        /*这里的files写的是需要监控的文件的位置*/
         files:[             
             "./Home/View/PC/**/*.html",
             "./public/group/css/*.css",
             "./public/group/js/*.js"
         ],
         logLevel: "debug",
         logPrefix:"insgeek",
         /*这里的proxy写的是需要代理的服务器，我自己的wamp启动的是localhost:80*/
         proxy:"localhost:80",
         ghostMode: {
	        clicks: true,
	        forms: true,
	        scroll: true
	     },
         /*这里写的是代理后，bs在哪个端口打开*/
         port: 81,
         /*这里设置的是bs运行时打开的浏览器名称*/
	     browser: "chrome"
    });
});
```
##### 2.2 启动静态服务器
```js
const gulp = require('gulp');
const browserSync = require('browser-sync').create();
const reload = browserSync.reload;
/*实时监控*/
gulp.task("watch", function() {
    browserSync.init({
        files: [
            "./work/*/*.html",
            "./work/*/*.css",
            "./work/*/*.js"
        ],
        logLevel: "debug",
        logPrefix: "insgeek",
        server: {
            /*这里写的是html文件相对于根目录所在的文件夹*/
            baseDir: "./work/statement"
            /*这里如果不写，默认启动的是index.html，如果是其他名字，需要这里写*/
            // index: "insurance_template_statement.html"
        },
        ghostMode: {
            clicks: true,
            forms: true,
            scroll: true
        },
        browser: "chrome"
    });
});
```
完成上述配置后，可以在cmd命令行切换到工作目录，运行gulp watch启动broserSync。

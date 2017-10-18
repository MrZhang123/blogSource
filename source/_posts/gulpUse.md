---
title: 基于Gulp的前端自动化工程搭建
date: 2016-09-07 23:25:06
tags: Javascript Gulp Nodejs
comments: true
categories: "Javascript Gulp Nodejs"
---
> 上个月月底在公司提出关于前后端分离的想法，并且开始研究关于前后端分离，前端工程化，模块化的一些东西，上周开始我准备自己开始写基于Gulp流的前端工程文件，这两天有时间，着手开始实现这个想法，但是写的过程中，遇到了一些问题，正是因为这些问题的解决让我对Gulp的流式处理有了更深的理解，写下这篇文章，分享一下这俩天我在写Gulp的时候学到的一些东西。

<!--more-->

## 准备工作

### 安装Node

首先Gulp是基于Nodejs的，所以安装Nodejs是前提，Node可以说是前端神器，基于Node有各种各样的工具，正是因为这些工具让我们非常方便的构建前端工程。

#### 更改Node插件默认安装位置（非必需）

我自己一般不喜欢在C盘状太多与系统无关的东西，而通过Node自带的npm安装的插件默认在C盘，但是我将Node安装到D盘后，想让插件就安装在Nodejs的主目录下，怎么办呢？

1. 在Node主目录下新建"node_global"及"node_cache"两个文件夹

2. 启动cmd，输入

```c
//后面的设置目录根据你的目录结构自行更改
npm config set prefix "D:\Program\nodejs\node_global"
npm config set cache "D:\Program\nodejs\node_cache"
```

3. 关闭cmd，打开系统对话框，“我的电脑”右键“属性”-“高级系统设置”-“高级”-“环境变量”。

4. 进入环境变量对话框，在系统变量下新建"NODE_PATH"，输入"D:\Program\nodejs\node_global\node_module"。 由于改变了module的默认地址，所以上面的用户变量都要跟着改变一下（用户变量"PATH"修改为"D:\Program\nodejs\node_global\"），要不使用module的时候会导致输入命令出现“xxx不是内部或外部命令，也不是可运行的程序或批处理文件”这个错误。

经过这四步的设置就可以让安装的Node插件放在Nodejs的主目录了。

### 安装Gulp

```c
//全局安装Gulp
npm install -g gulp
//在项目中安装Gulp
npm install --save-dev gulp
```

运行`gulp -v`,如果不报错，表示安装成功

然后在命令行运行

```c
npm init
```

让项目生产`package.json`文件

## 搭建工程

众所周知，在开发工程中有开发和上线两个过程，在开发中，我们一般需要自动刷新以及实时编译，但是如果上线，我们就需要考虑很多优化的东西，比如文件编译压缩，静态资源放缓存处理等等问题，我自己搭的这个工程只涉及到文件编译压缩，实时刷新，静态资源放缓存这三个基本的流程。

在项目的目录结构如下

```c
-------------------project
    |
    |--------------dist (该文件夹为打包生成的)
    |   |
    |   |----------css
    |   |   |
    |   |   |------index-9dcc24fe2e.css
    |   |
    |   |----------js
    |   |   |
    |   |   |------index-9dcc24fe2e.js
    |   |----------index.html 
    |
    |--------------src
    |   |
    |   |----------scss
    |   |   |
    |   |   |------index.scss
    |   |
    |   |----------js
    |   |   |
    |   |   |------index.js
    |   |
    |   |----------index.html
    |--------------gulpfile.js
    |--------------package.json
```

### 开发所用流程

#### 文件编译

在工程中准备使用scss作为css的预编译，所以需要利用gulp对scss进行编译，所以首先安装gulp-sass。

```c
npm install --save-dev gulp-sass
```

安装完成之后，直接在gulpfile.js引用配置

```js
const sass = require('gulp-sass'); //scss编译

gulp.task('scss:dev',()=>{
    gulp.src('src/scss/*.scss')
    .pipe(sass())
    .pipe(gulp.dest('dist/css')); //将生成好的css文件放到dist/css文件夹下
});
```

这里简单介绍下gulp的两个api：

gulp.src()输入符合所提供的匹配模式或者匹配模式的数组的文件。将返回一个stream或者可以被piped到别的插件中。**读文件**

gulp.dest()能被pipe进来，并且将会写文件。并重新输出（emits）所有数据，因此可以将它pipe到多个文件夹，如果文件夹不存在则将会自动创建。**写文件**

#### 实时刷新

实现实时刷新的工具有很多，我自己使用browser-sync，这个工具的功能非常强大，想了解它更多的用法可以查看官网：[http://www.browsersync.cn/](http://www.browsersync.cn/)。

首先我们在项目中安装该模块

```c
npm install --save-dev browser-sync
```

根据官网的browser-sync与gulp的配置，得到如下配置：

```js
const browserSync = require('browser-sync').create(); //实时刷新
const reload = browserSync.reload;

gulp.task('dev',['scss:dev'],function () {
    browserSync.init({
        server:{
            baseDir:'./'  //设置服务器的根目录
        },
        logLevel: "debug",
        logPrefix:"dev",
        browser:'chrome',
        notify:false //开启静默模式
    });
    //使用gulp的监听功能，实现编译修改过后的文件
    gulp.watch('src/scss/*.scss',['scss:dev']);
    gulp.watch(('*.html')).on('change',reload);
});
```


这样，一个简单的gulp开发流程就出来了，仅仅只是一个编译scss和一个实时刷新。

### 打包上线所有流程

打包上线，我们更多的是考虑，静态资源防缓存，优化。对css，js的压缩，对图片的处理，我写的这个简单的流程中并没有涉及对图片的处理，所以这里仅针对css，js，html处理。

压缩css我们使用gulp-sass就可以，因为它在编译scss的时候有一个配置选项可以直接输出被压缩的css。压缩js我使用了gulp-uglify，静态资源防缓存使用gulp-rev和gulp-rev-collector。

#### 对css，js的处理

```js
//scss编译
gulp.task('css',()=> {
    gulp.src('src/scss/*.scss')
        .pipe(sass({
            outputStyle: 'compressed'               //编译并输出压缩过的文件
        }))
        .pipe(rev())                                //给css添加哈希值
        .pipe(gulp.dest('dist/css'))
        .pipe(rev.manifest())                       //给添加哈希值的文件添加到清单中
        .pipe(gulp.dest('rev/css'));
});
//压缩js
gulp.task('js', ()=> {
    gulp.src('src/js/*js')
        .pipe(uglify())
        .pipe(rev())                                //给js添加哈希值
        .pipe(gulp.dest('dist/js'))
        .pipe(rev.manifest())                       //给添加哈希值的文件添加到清单中
        .pipe(gulp.dest('rev/js'));
});
```

其中gulp-rev是为css文件名添加哈希值，而rev.manifest()会生成一个json文件，这个json文件中记录了原文件名和添加哈希值后的文件名的一个对应关系，这个对应关系在最后对应替换html的引用的时候会用到。

生成的json文件如下：

```json
{
  "index.css": "index-9dcc24fe2e.css"
}
```

由于给文件添加了哈希值，所以每次编译出来的css和js都是不一样的，这会导致有很多冗余文件，所以我们可以每次在生成文件之前，先将原来的文件全部清空。

gulp中也有做这个工作的插件---gulp-clean，因此我们可以在编译压缩添加哈希值之前先将原文将清空。

#### 清空生成的项目文件

```js
const clean = require('gulp-clean');                 //清空文件夹里所有的文件
//每次打包时先清空原有的文件夹
gulp.task('clean', ()=> {
    gulp.src(['dist', 'rev'], {read: false}) //这里设置的dist表示删除dist文件夹及其下所有文件
        .pipe(clean());
});
```

#### 让添加哈希编码的文件自动添加到html中

前面提到的gulp-rev实现了给文件名添加哈希编码，但是在打包完成后如何让原来未添加哈希值的引用自动变为已经添加哈希值的引用，这里用到gulp-rev的一个插件gulp-rev-collector，配置如下：

```js
//将处理过的css，js引入html
gulp.task('reCollector',()=>{
    gulp.src(['rev/**/*.json','src/*.html'])
        .pipe(reCollector({
            replaceReved: true,  //模板中已经被替换的文件是否还能再被替换,默认是false
            dirReplacements: {   //标识目录替换的集合, 因为gulp-rev创建的manifest文件不包含任何目录信息,
                'css/': '/dist/css/',
                'js/': '/dist/js/'
            }
        }))
        .pipe(gulp.dest('dist'))
});
```

##### 并没有正常替换？

在我自己写的时候，出现这个问题，运行完成该任务后，html中的css和js引用并没有发生变化，网上搜了半天，才知道是由于自己用了gulp-rename插件，然后将文件名都添加了.min（至于为什么添加，仅仅是因为是压缩过的，应该写个）而在自己写的html里面引用的文件并没有.min，由于gulp-rev-collector在替换的时候根据生成的json文件替换，在json中，文件都有了.min而在html中没有，所以无法匹配，自然也就不能实现替换了，所以在替换的时候一定要注意gulp-rev生成的json文件中的css，js与html中的引用的一样，否则无法实现替换。

<font color="red">在gulp-rev-collector的api中有一个revSuffix，这个看起来可以实现类似于gulp-rename的功能，但是不知道该怎么用，大家如果知道的话请告诉我...</font>

#### 执行所有任务

完成上面几个步骤后我们将所有任务串起来，让其可以一条命令然后全部执行

```js
gulp.task('build',['clean', 'css', 'js', 'reCollector']);
```

#### 再次理解gulp

##### gulp---它的task是顺序执行吗？

本以为到这里，就算是写完了，运行，完美，打包生成文件，再运行一次，报错了！！！！

```c
[19:04:57] Finished 'default' after 7.38 μs
stream.js:74
      throw er; // Unhandled stream error in pipe.
      ^

Error: ENOENT: no such file or directory, stat 'D:\project\dist\js\index-6045b384e6.min.js'
    at Error (native)
```

提示我找不到这个文件，这让我很郁闷啊，然后我分开执行，很ok，可以确定是执行顺序有问题，很可能在没有清理完成就执行后面了，查了gulp的官网文档才知道**本身gulp的pipe是一个一个任务进行的，是同步的，但是每个task之间是不同步的，是一起进行的**，这也验证了我的猜想，所以在网上找如何解决这个问题，找到一个叫run-sequence的npm插件，配置文件如下：

```js
//进行打包上线
gulp.task('build', ()=> {
    runSequence('clean', ['css', 'js'], 'reCollector');
});
```

本以为运行就ok，结果，还是报错，这里就涉及到对gulp的另一个理解

##### run-sequence插件对异步任务的处理

在用这个插件让任务有序进行后，我想进一步直观的看到它对任务的序列化，自己写了一个demo，如下：

```js
gulp.task('a',function(){
    setTimeout(function () {
        console.log(1);
    },30);
});
gulp.task('b',function() {
    console.log(2);
});
gulp.task('ab',function(){
    runSequence('a','b');
});
```

但是这里就出现问题了，runSequence不管用了，找插件的说明和gulp官方文档，原来异步任务，像setTimeout，readFile等，需要添加一个callback的执行，这里的callback()就会返回一个promise的resolve()，告诉后面的任务，当前任务已经完成，后面可以继续执行了，所以在task a里面执行callback。

```js
gulp.task('a',function(cb){
    setTimeout(function () {
        console.log(1);
        cb();
    },30);
});
```

那为什么前面写的那些任务不需要添加一个callback呢？由于gulp的pipe流让每一个task中的小任务（每一个pipe）顺序执行，从而整个pipe流是同步的，并不是异步任务，所以并不需要手动让其返回promise，run-sequence会自动帮我们管理。

## gulpfile的分离

在前面我们将dev和build写在了一个叫gulpfile的文件中，虽然可以执行，但是当我们的工程越来越大的时候，会导致gulpfile可维护性降低，那能否让dev和build分别写在两个文件中呢？答案是可以的，我们可以新建两个文件，分别为gulpfile-dev.js和gulpfile-build.js，其实我们在运行gulp build的时候，其实是运行了gulp --gulpfile gulpfile.js build，前者相当于后者的缩写，所以在运行gulp的时候在命令中输入如下：

```c
gulp --gulpfile gulpfile-dev.js

gulp --gulpfile gulpfile-build.js
```

就可以在gulp运行时候指定gulpfile。这样我们再原来的task中的buil和dev改成default就可以直接运行以上命令达到预期效果。

但是每次敲这么长的命令很烦，怎么办呢？我们可以在package.json的scripts中添加如下json：

```json
"dev": "gulp --gulpfile gulpfile-dev.js"
"build": "gulp --gulpfile gulpfile-build.js"
```

这样，我们在运行的时候，直接在命令行输入：

```c
npm run dev
npm run build
```

就可以实现打包了，是不是很酸爽，哈哈！

## 总结

至此，我们就完成了一个简易的基于gulp的前端工程的搭建，很多东西确实，想着并不难，做起来会出现各种各样意想不到的问题，gulp很早就知道，都是单个任务在写，然后用哪个执行哪个命令，直到自己写完这个这个简单的工程，才对gulp有了更深入的理解。

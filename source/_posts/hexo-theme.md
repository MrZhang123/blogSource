---
title: 如何写一个自己的hexo主题
date: 2017-04-01 20:57:48
tags: hexo
comments: true
categories: "hexo"
---

>好长时间不写东西了，都在忙工作，去年三月份的时候我搞了一个hexo的博客，主题换过两三个吧，感觉都大部分的hexo主题都是东西比较多，有那么两三个比较简单的，但是自己并不是很喜欢，所以去年的时候就想着自己抽时间写一个自己的主题，但是当初看了几个hexo主题的源码，完全看不懂是什么，所以放弃了，今年又定了一个目标，写一个自己的hexo主题，终于终于在文章[写一个自己的Hexo主题](https://segmentfault.com/a/1190000006057336])和该文中提到的作者自己写的主题[hexo-showonne](https://github.com/showonne/hexo-showonne)的启发下，开始了自己的hexo主题之旅，之后更参考自己用了很长时间的hexo主题——[hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus)进行了修正。

<!--more-->

我的博客就是用的我写的主题，博客地址：https://mrzhang123.github.io/

项目地址：https://github.com/MrZhang123/hexo-theme-SpecLumn


## 搭建项目


参考大多数的hexo主题的目录结构不难发现，大部分都会有layout、source以及配置文件_config.yml，所以具体目录如下：

```sh
├── _config.yml      #主题的配置文件
├── layout          #布局模版文件夹 
│   ├── comment     #评论相关模版
│   ├── common      #公共的模版
│   │   └── post    #post页相关模版
│   └── plugin      #插件
├── source          #主题的css和js
│   └── css
│       └── images  #主题中用到的图片
└── source_src      #scss所在的文件夹
    └── scss
        ├── code-style
        └── markdown-style
```

项目基于gulp搭建，hexo默认使用stylus，但是我使用的scss，然后生成css，但是在项目过程中发现一个问题（同时明白为什么hexo默认使用stylus,可以直接识别styl文件），hexo提供了`hexo-config`方法，可以读取`_config.yml`中的配置。所以如果想要在`_config.yml`中动态设置代码的高亮主题，可以在stylus中写如下代码：

```css
/*
一个三元表达式设置默认主题
然后在index.styl中设置引入的表达式
*/
highlight = hexo-config("customize.highlight") || "tomorrow"

@import "highlight/" + highlight
```
但是我在项目中使用的是gulp-sass编译成的css文件，无法实现根据_config.yml动态的引入文件，所以准备下次改的时候直接换stylus（虽然hexo可以通过装插件识别scss，但是自己测试并未成功）。

## 布局

### 布局文件layout.ejs

在layout文件夹下，布局被写在layout.ejs中，由于hexo支持**模块儿化布局**，使用hexo提供的局部函数`partial`载入其他模版文件，配合ejs的语法，布局文件layout.ejs：

```js
<%- partial('common/head') %>
<div class="content">
    <%- partial('common/header') %>
    <div class="main">
        <%- body %>
    </div>
</div>
<%- partial('common/footer') %>
<%- partial('common/foot') %>
</body>
</html>
```

hexo默认使用ejs语法，但是可以通过安装插件使用Haml和Jade（Jade以前看过，并不喜欢缩进的写法，所以在开发工程中也没用使用同样使用缩进语法的stylus）。

项目中直接使用了ejs，所以这里简要列一下ejs中tag的使用：

- `<%`' 脚本标签，用于控制流程，不会输出
- `<%=` 将转义的值输出到模版
- `<%-` 将未转义的值输出到模版
- `<%#` 注释标签
- `<%%` 输出 '<%'
- `%>`  闭合标签
- `-%>` Trim-mode ('newline slurp') 标签, 移除随后的换行符

### 其他模版

| Url        | Description   |  Layout  |
| --------   | -----:  | :----:  |
| /     | 首页      |   index.ejs     |
| /yyyy/mm/dd/:title/        |   文章   |   post.ejs   |
| /archives/        |    归档    |  archive.ejs  |
| /tags/:tagname/        |    某个标签的归档    |  tag.ejs  |
| /:else/   |    其他    |  page.ejs  |

在我的主题中只有主页index.ejs，文章页post.ejs。

## 变量

模版中获取文章，调用配置项等会使用到变量，hexo提供了很多变量供我们使用（[hexo提供的变量](https://hexo.io/zh-cn/docs/variables.html)），其中比较重要的有：

- `page`：针对该页面的内容以及 front-matter 所设定的变量
- `config`：网站配置（hexo的_config.yml）
- `theme`：主题配置（theme的_config.yml）
- `url`：当前页面的完整网址
- `path`：当前页面的路径（不含根路径）

另外在参看主题hexo-theme-icarus的源码发现一些变量的使用，但是我在官方文档中并没有找到。

- `post.title`：文章的题目
- `post.excerpt`：文章的摘要，即写文章的时候`<!--more-->`之上的那段儿，首页显示
- `post.comments`：评论模块
- `post.photots`：文章中插入的图片
- `post.content`：文章内容
- `post.date`：文章的时间

以上均获取的是hexo生成的markdown文件中的配置。

## 辅助函数

在hexo中同样提供了很多[辅助函数](https://hexo.io/zh-cn/docs/helpers.html)方便在博客主题中使用，但是同样很多都用不着，常用的有：

- `url_for`：输出路径
- `partial`：载入其他模版文件
- `css`：载入css文件
- `js`：载入js文件
- `data_xml`：插入 XML 格式的日期
- `paginator`：生成分页，其中可以插入配置，partial一样，例如改变上一页下一页的文字，可以配置`{ prev_text: '« 上一页', next_text: '下一页 »'}`

关于`partical`需要注意几点

1. 它可以添变量，在被引用文件直接可以饮用，例如：

```js
/*引用*/
<%- partial('plugin/scripts', { isHead: true }) %>
/*被引用文件判断*/
<% if (typeof(isHead) !== 'undefined' && isHead) { %>

<% } %>
```

2. 可以添加class名字

```js
<%- partial('post/date', { class_name: 'article-date', date_format: null }) %>
```

## 其他

### 评论

hexo中添加评论只需要在ejs中添加相应的`script`标签插入代码即可，以前的时候用多说，现在多说要关闭了，准备换个，但是找来找去没找到一个比较适合的，同事介绍有一个叫来必力的还不错，试用确实觉得可以，可以替代多说，所以推荐一下。[官方网站](https://livere.com/)。

### 代码高亮

代码高亮其实就是引入一段css的代码，为了实现可以通过配置文件动态引入，就需要配合`hexo-config`，前面已经提到过，这里不再重复。

## 最后

最后写一点自己的想法，说实话，hexo的官方文档，实在是烂的没话可说，跟webpack1的文档不相上下，估计连作者自己都不一定看得懂，所以要想自己写一个主题的同学们还是在github参考一下写的比较好的主题的源码吧。

另外，当前这个版本功能相对简单一点儿而且在后期的修改过程中发现用scss确实不如直接用stylus方便一些，所以准备在以后改用stylus。

## 参考

[hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus)
[写一个自己的Hexo主题](https://segmentfault.com/a/1190000006057336)
[hexo-showonne](https://github.com/showonne/hexo-showonne)
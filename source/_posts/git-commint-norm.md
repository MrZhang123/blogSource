---
title: Git Commint规范
date: 2017-10-18 22:11:36
tags: git&github
comments: true
categories: "git&github"
---

>之前看一个github的开源项目，看到要求使用angular的git commit规范，刚好自己项目中也需要规范git commit，所以就研究总结一下。

<!--more-->

## 配置检测git commit是否合法

```sh
npm install --save-dev @commitlint/cli husky
```

新建`commitlint.config.js`配置git提交规范（这里使用的是angular的提交规范）

```js
module.exports = {
  rules: {
    'body-leading-blank': [1, 'always'], //body开始于空白行
    'body-tense': [1, 'always', ['present-imperative']], 
    'footer-leading-blank': [1, 'always'], //footer开始于空白行
    'footer-tense': [1, 'always', ['present-imperative']],
    'header-max-length': [2, 'always', 72],
    'scope-case': [2, 'always', 'lowerCase'], //scope小写
    'subject-empty': [2, 'never'], //subjec不t为空
    'subject-full-stop': [2, 'never', '.'], //subject结尾不加'.'
    'subject-tense': [1, 'always', ['present-imperative']], //以动词开头，使用第一人称现在时，比如change，而不是changed或changes
    'type-case': [2, 'always', 'lowerCase'], //type小写
    'type-empty': [2, 'never'], //type不为空
    'type-enum': [
      2,
      'always',
      [
        'build',
        'chore',      //构建过程或辅助工具的变动
        'docs',       //文档（documentation）
        'feat',       //新功能（feature）
        'fix',        //修补bug
        'perf',
        'refactor',   //重构（即不是新增功能，也不是修改bug的代码变动）
        'revert',
        'style',      //格式（不影响代码运行的变动）
        'test',       //增加测试
      ],
    ], //type关键字必须是其中之一
  },
};
```

配置husky

```json
{
  "scripts": {
    "commitmsg": "commitlint -e"
  }
}
```

配置完成之后，提交所写的commit如果不符合规范，则无法进行下一步操作

## Commit message 的格式

每次提交，Commit message 都包括三个部分：Header，Body 和 Footer

```xml
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

Header是必需的，Body和Footer可以省略

### header

Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）

#### type

该字段用于说明commit的类型，被指定在上面的`type-enum`中

```js
'build',
'chore',      //构建过程或辅助工具的变动
'docs',       //文档（documentation）
'feat',       //新功能（feature）
'fix',        //修补bug
'perf',
'refactor',   //重构（即不是新增功能，也不是修改bug的代码变动）
'revert',
'style',      //格式（不影响代码运行的变动）
'test',       //增加测试
```

如果type为feat和fix，则该 commit 将肯定出现在 Change log 之中。其他情况建议是不要放入

#### scope

用于说明commit的影响范围

#### subject

subject是 commit 目的的简短描述，不超过50个字符

要求如下：

* 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
* 第一个字母小写
* 结尾不加句号（.）

### body

body 部分是对本次 commit 的描述，可以分成多行，例如：

```sh
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点:

1. 使用第一人称现在时，比如使用change而不是changed或changes

2. 应该说明代码变动的原因，以及跟以前提交的对比

### footer

footer只用于以下两种情况：

#### 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法

```sh
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

#### 关闭Issue

如果当前 commit 针对某个issue，那么可以在 footer 部分关闭这个 issue

```sh
//关闭一个
Closes #1
//关闭多个
Closes #2, #3, #4
```

### revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，必须以revert:开头，后面跟着被撤销 commit 的 header

```sh
revert: refactor(compiler): introduce `TestBed.deprecatedOverrideProvider` (#19558)

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的hash是被撤销 commit 的 SHA 标识符

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的Reverts小标题下面。

### Example

```sh
refactor(compiler): introduce `TestBed.deprecatedOverrideProvider` (#19558)

This allows use to fix `TestBed.overrideProvider` to keep imported `NgModule`s eager,
while allowing our users to still keep the old semantics until they have fixed their
tests.

PR Close #19558
```

## 生成change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成。

生成的文档包括以下三个部分。

```sh
New features
Bug fixes
Breaking changes.
```

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

conventional-changelog 就是生成 Change log 的工具，运行下面的命令即可。

```sh
$ npm install -g conventional-changelog
$ cd my-project
$ conventional-changelog -p angular -i CHANGELOG.md -w
```

上面命令不会覆盖以前的 Change log，只会在CHANGELOG.md的头部加上自从上次发布以来的变动。

如果你想生成所有发布的 Change log，要改为运行下面的命令。

```sh
$ conventional-changelog -p angular -i CHANGELOG.md -w -r 0
```

为了方便使用，可以将其写入package.json的scripts字段。

```json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  }
}
```

以后，直接运行下面的命令即可。

```sh
$ npm run changelog
```

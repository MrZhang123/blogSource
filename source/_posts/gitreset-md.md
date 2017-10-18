---
title: 使用git回退远程库代码
date: 2017-03-04 20:57:48
tags: Git
comments: true
categories: "Git"
---

> 前几天在公司合代码的时候出现了问题，需要reset回去，在同事的帮助下利用git的log进行了reset，虽然很简单，但是还是想记录一下。

<!--more-->

## 本地代码回滚

首先需要使用`git log`查看需要回退的hash码，但是因为是多人合作，所以我在gitlab中切换到自己的分支，然后在`Graphs->Network`中查看树桩gitlog图，在其中有每一次git操作的hash码，如果想回退到某个git操作，只需要进行如下操作：

```shell
$ git reset --hard "commit id"
```
即可回退到当前提交，这次提交之后的所有提交的log也会随着回退而全部消失。

但是这样的回退只是本地代码回退，远程代码库依然是已经更新过的，接下来需要让远程代码库更新。

### reset & revert

在git的命令中，这两个命令都是回退，不同的在于，`reset`在回退后，回退点之前的记录也会被清空，但是`revert`会保留，所以回退的时候使用哪一个可以自己选择。

这里有两个命令需要区别一下，

## 更新远程代码库

因为reset之后本地库落后于远程库一个版本，因此需要强制提交。

```shell
$ git push origin master -f
```

这里的`-f`可以强制将本地代码库提交到远程。

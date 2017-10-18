---
title: Linux日常使用总结
date: 2016-10-02 23:33:47
tags: linux
comments: true
categories: "linux"
---
> 自己一直对linux充满好奇，这次换了新的工作后，由于使用Mac OS和linux，所以国庆回来，给自己的电脑装了ubuntu来熟悉linux。在安装完成后遇到一些问题，经过谷歌都一一解决了，写这篇博文分享给大家。

<!--more-->

## 1．设置获取root权限

```c
sudo passwd root //设置root密码
```

然后输入当前系统账户的密码并设置新的UNIX密码，密码更新成功后在终端输入`su`然后输入刚刚设置好的新的UNIX密码，即可进入root权限，如果想退出，输入`exit`。


## ２．在linux下如何安装shadowsocks

因为自己是搞开发的，而且非常喜欢谷歌，所以就需要一个自由的网络环境，博主用的是shadowsocks，在windows下，有客户端，但是现在装了linux，既然装了linux，能用命令行搞定的就尽量用命令行，所以我选择在终端安装shadowsocks客户端，步骤如下：

#### 安装shadowsocks客户端

```c
sudo apt-get update 
sudo apt-get install python-gevent python-pip
pip install shadowsocks
```

#### 建立配置文件

打开终端，运行`vim /etc/ss.json`，然后写入json

```json
{
	"server" : "you server",
	"server_port" : 0000,
	"local_address" : "127.0.0.1",
	"local_port" : 1080,
	"password" : "you password",
	"method" : "aes-256-cfb",   //shadowsocks的加密方式
	"fast_open" : false
}
```

#### 开启shadowsocks

```c
sslocal -c /etc/ss.json

// 开启后显示以下内容，代表开启成功：
// INFO  loading libcrypto from libcrypto.so.1.0.0
// INFO  starting local at 127.0.0.1:1080
```
#### 设置开机启动

```c
// 打开图形化开机启动项管理界面
gnome-session-properties
// 添加(Add) -> 名称(name)和描述(comment)随便填，命令(Command)填写如下： 
sslocal -c /etc/ss.json
```

#### 火狐浏览器配置

设置→首选项→高级→网络→链接→设置→手动配置代理→socks主机：127.0.0.1 端口：1080→确定

#### chrome配置

在终端中输入如下命令：

```c
chromium-browser --proxy-server=socks5://127.0.0.1:1080
```

进入后安装SwitchyOmega，然后配置此插件：

情景模式→删除原有的情景模式，新建情景模式（原有的情景模式无socks代理）→手动配置→SOCKS代理：127.0.0.1 端口1080→保存 →ok

##### note:

我在这里运行这个命令后依然无法进入，不清楚是怎么回事儿，所以我直接找了个host进行了替换，关于host，我推荐[老d博客中提供的](https://laod.cn/hosts/2016-google-hosts.html)，具体用法他博客中有写。然后才进入的应用商店下载好。

## ３．安装相应的软件

#### 安装Gdebi软件包安装程序

在ubuntu中有自己的软件中心，但是有些软件在里面找不到，需要自己去官网下载客户端然后安装（比如网易云音乐，搜狗拼音等），下载好的安装包大部分都是`.deb`文件，查阅了需要资料，安装这个用`gdebi`比较方便，在软件中心搜索`Gdebi软件包安装程序`安装就好，但是，既然用linux了，就用一下它强大的终端，所以我说一下在终端如何安装

在终端输入

```c
sudo apt-get install gdebi
```

但是我在这里输入后，提示我缺少相应的依赖，并让我运行

```c
sudo apt-get install -f
```

运行完成这个命令后，相应的依赖就被安装好了，这里的这个命令主要是修复依赖关系（depends）的命令，之后重新运行`sudo apt-get install gdebi`即可。

#### 安装对应的软件

在安装完成这个后，安装软件就方便很多，我比如安装chrome浏览器，直接进入下载好文件的目录，运行

```c
sudo gdebi chrome.deb //这里是你要安装包的名字
```

完成后去搜索即可。<font color='red'>在安装时候我发现有的软件在安装完成后需要重启下电脑才会有。</font>

#### 卸载apt-get安装的东西

有时候需要卸载一些`apt-get`安装的东西，需要命令为

```c
sudo apt-get remove XXX //XXX为要卸载的东西
```

#### 安装node

做前端开发，node是必不可少的，所以需要安装一下，在windows下都是直接下载安装包安装，我本来也在linux下下了个安装包安装了，但是查了一下发现用nvm安装node的比较多，而且那天在公司老大也说在mac下推荐使用nvm安装，所以自己决定试下。

##### 1.安装 nvm 之后最好先删除下已安装的 node 和全局 node 模块：

```c
npm ls -g --depth=0 //查看已经安装在全局的模块，以便删除这些全局模块后再按照不同的 node 版本重新进行全局安装
sudo rm -rf /usr/local/lib/node_modules //删除全局 node_modules 目录
sudo rm /usr/local/bin/node //删除 node
cd  /usr/local/bin && ls -l | grep "../lib/node_modules/" | awk '{print $9}'| xargs rm //删除全局 node 模块注册的软链
```

话说最后一条命令运行会报rm使用错误，不知道怎么弄...

##### 2.安装[nvm](https://github.com/creationix/nvm)

运行以下两条命令之一

```c
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.0/install.sh | bash
//or
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.32.0/install.sh | bash
```

然后进入(~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc)这些文件，看哪个文件有，在最后一行添加.

```c
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```

这里如果不太会改这个的话，可以设置终端从而达到效果，在终端打开`编辑→配置文件首选项→命令`中勾选`以登录shell方式运行命令`。在ubuntu中没有`执行命令时更新登录登录记录`，如果有，一起勾选，然后在终端输入

```c
nvm
```

即可看到帮助，之后运行

```c
nvm install node
```

就可以安装到最新版本的node，更多关于nvm的用法可参考[node版本管理工具nvm-Mac下安装及使用](https://segmentfault.com/a/1190000004404505)

至此，linux下常用的软件就安装完成...

---
title: Mac oh-my-zsh在Pycharm、VSCode下乱码和颜色问题解决
date: 2018-09-10 15:34:41
tags:
	- terminal
	- Pycharm
	- VSCode
categories: 技术
---

Mac下的oh-my-zsh主题在vscode和pycharm的终端调用里出现了问题：

1. vscode中箭头变成了乱码。
2. pycharm里的配色不对，导致看不清字体。

#### 问题1
参考了[柳婼的博客](https://blog.csdn.net/liuchuo/article/details/79967960)，解决方法是加载一套支持非ASCII编码的字体。

```
> git clone https://github.com/powerline/fonts.git

> cd fonts

> ./install.sh
```

安装好后，进入terminal，选择`偏好设置->文本->字体更改`，把字体改为刚刚加载的字体（在用户集合中）。

重新打开vscode的terminal即可。

#### 问题2

发现在其他环境下没有类似问题，应该为pycharm的颜色配置问题，主要是黑白颜色反了。

打开Pycharm的偏好设置，搜索`color`关键字，打开`console colors`,里面有`ANSI colors` ,查看相应名称的颜色是否对应正确。发现黑白颜色反了，调换过来就好。

问题的原因暂未可知，有可能是以前安装其他颜色主题的时候搞错了。

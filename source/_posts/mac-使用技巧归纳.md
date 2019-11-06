---
title: mac 使用技巧归纳
date: 2019-03-17 13:23:27
categories: "其他"
tags:
    - mac
---

在家用 mac，总会有时候遇到快捷键与 windows 不同，或者其他的使用技巧，这里做个简单地归纳吧，方便自己的使用与查找。
时不时会做个更新。

<!-- more -->

# 百度云下载破解

[Mac 百度云下载限速破解教程（附软件）](https://blog.csdn.net/cheneykl/article/details/80941005)

# 截图

局部截图：Command + shift + 4

截当前窗口：Command + shift + 4, 点击空格键

全屏截图：Command + shift + 3

touch bar 截图： Command + shift + 6

# 录屏

系统自带的 QuickTime Player 就有录屏功能

![QuickTime Player](/images/quickTime_Player.png)

# 好玩的读单词

在终端中输入： say + 单词，就会读出这个单词

# 使用Finder搜索大文件

![使用Finder搜索大文件](/images/使用Finder搜索大文件.jpg)

# 便签

1. 系统自带的便笺

2. simple antnotes：比系统的好用

# 调整 LaunchPad 图标数量及大小

可以根据需要让每页显示更多或更少应用

打开终端
1、调整每列图标数量，把m换成你想要的数字（推荐5~7）
defaults write com.apple.dock springboard-rows -int m
2、调整每行图标数量，把n换成你想要的数字（推荐6~11）
defaults write com.apple.dock springboard-columns -int n
3、修改后重置 Launchpad
defaults write com.apple.dock ResetLaunchPad -bool TRUE;killall Dock

# 用快捷键快速移动光标

command + 方向键：将光标移至最顶端/末端
option + 方向键：以单词为单位移动光标（中文会自动识别词组）

# 聚焦功能

快速查找电脑上的内容，也可以搜索互联网上内容

打开聚焦：Command + 空格

搜索互联网：打开聚焦后，输入关键字，再 Command + b， 会打开默认浏览器就行搜索

搜索汇率：数字 + 单位，如 100rmb

# 复制粘贴等快捷键有时会失效

有时会遇到第一次复制或粘贴时，不起作用，需要多次复制粘贴才起作用。网上推测是 [三指拖动的锅](https://www.zhihu.com/question/35784714), 我觉得也是。
有些网友是因为装了 popclip 导致的。而我检查我的 mac 后，发现是我的有道词典开启了划词释义，关闭后就好了。

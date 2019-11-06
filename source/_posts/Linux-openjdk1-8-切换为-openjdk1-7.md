---
title: openjdk1.8 切换为 openjdk1.7
date: 2019-01-31 20:47:57
categories: "Linux"
tags:
    - linux
    - openjdk
    - ppa
---

# oracle openjdk ppa source

工作需要编译 Android5.1源码，需要 openjdk1.7 的环境。而工作站上是 openjdk1.8，公司网络又有各种限制，特此分享 oracle openjdk ppa source 的安装切换版本的方式。

```
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-7-jdk

sudo update-alternatives --config java
sudo update-alternatives --config javac
```

注：
add-apt-repository 是由 python-software-properties 这个工具包提供的
所以要先安装python-software-properties 才能使用 add-apt-repository
否则会显示“command not found”
安装方法：apt-get install python-software-properties
sudo apt-get install software-properties-common
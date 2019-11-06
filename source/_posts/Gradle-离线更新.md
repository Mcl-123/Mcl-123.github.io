---
title: Gradle 离线更新
date: 2019-02-14 22:13:29
categories: "Android"
tags:
    - gradle
---

遇到下载的项目，Android studio 一直无法下载成功对应的 gradle 版本，所以选择手动下载 gradle 对应的 zip 文件。

1. 先到 [Gradle 官网](http://services.gradle.org/distributions/)下载对应的版本 complete 包（all）到本地。

![gradle_download](/images/gradle_download.png)

2. 拷贝 gradle-x.x-all.zip 到该目录下:

![gradle_save_location](/images/gradle_save_location.png)

注意: 1.如果在dists目录下有该gradle版本的文件夹，则拷贝压缩包到该版本文件夹的随机码路径文件夹下。
    : 2.否则自己按照之前的对比新建一个文件夹（例如：gradle-4.6-all），然后重新建一个随机码作为名字的文件夹，将gradle-4.6-all.zip拷贝到该目录下。
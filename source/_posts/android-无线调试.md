---
title: android 无线调试
date: 2019-10-08 21:35:32
categories: "工具"
tags:
    - Android Wifi ADB
---

使用 Android Wifi ADB 插件

![Android Wifi ADB](/images/Android_Wifi_ADB.png)

1. 安装以上插件,并重启 AS
2. 将手机连接至电脑,并确保电脑和手机在同一个网络下
3. 打开命令行,输入 adb tcpip 5555

然后就可以进行远程调试了.

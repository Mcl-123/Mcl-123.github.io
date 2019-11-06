---
title: android studio 无法正常启动
date: 2019-02-13 19:27:30
categories: "Android"
tags:
    - Android Studio
---

手贱！公司电脑升级了最新的 Android Studio，然后启动后就一直停在这个页面上：

![android studio 无法正常启动](/images/android_studio_无法正常启动.jpeg)

终止进程，重启电脑都试了，没用！

百度后有几种解决方案：

1. 说没配置好环境变量的（JDK 和SDK）。 可我已经配好了，pass。

2. 说内存不足，修改studio64.exe.vmoptions 文件 

<!-- more -->

```
-server

-Xms256m

-Xmx768m
-XX:ReservedCodeCacheSize=240m
-XX:+UseConcMarkSweepGC
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djna.nosys=true
-Djna.boot.library.path=
```

不起作用，pass。

3. 打开Android Studio的安装目录下的bin目录，打开idea.properties文件，在最后一行添加“disable.android.first.run=true”。
清除android studio的缓存cache，找到文件夹：C:\Users\pvwav\.AndroidStudio3.1\system\caches 清空所有，或者删除caches文件夹

起作用！ OK.

浪费了很多时间各种试，以后绝不手贱随意升级最新版了！

---
title: android 源码下载及编译
date: 2019-03-24 18:40:50
categories: "Android"
tags:
    - 源码
    - repo
---

# Android 源码下载

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
# 如果上述 URL 不可访问，可以用下面的：
# curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo
chmod a+x ~/bin/repo

mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY

# 初始化仓库： 
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest
# 如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
# REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'

# 如果需要某个特定的 Android 版本（Android 版本列表）：
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-4.0.1_r1

# 同步源码树（以后只需执行这条命令来同步）：
repo sync
```

# Android 源码编译

[自己动手编译Android源码(超详细)](https://www.jianshu.com/p/367f0886e62b)

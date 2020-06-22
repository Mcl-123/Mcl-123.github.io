---
title: Vultr 搭建 shadowsocks 教程
date: 2019-03-19 23:35:45
categories: "工具"
tags:
     - vultr
     - shadowsocks
     - 科学上网
---

连不了外网，上不了谷歌的感觉很不爽，咳咳，程序猿查资料而已。网上很多教程都被和谐了，生怕以后找不到教程，所以来这记录下。
仅供参考。
科学上网。

<!-- more -->

有很多 vps 提供商，例如 vultr，搬瓦工，linode 等，价格也不等，有特价的时候只要2.5美元一个月。我目前在用的是vultr。
vultr 的官网可以不翻墙就访问，而类似搬瓦工等有的就不行。（Uzer 内嵌的火狐浏览器之前也是可以免费翻墙，现在不行了）。
架设一台 vps 成本很低，如果想换节点，随时可以关机销毁，然后再重新搭建新的 vps。

下面介绍如何基于 vultr 搭建 shadowsocks 来科学上网。

# 搭建服务器

1. 登录[Vultr](https://www.vultr.com/?ref=7330907), 并注册账号。

2. 点击左侧 Billing, 选择支付，可以通过支付宝或微信支付。
     ![支付](/images/支付vps.jpg)

3. 点击左侧 Servers, 再点击右上角加号，创建服务器。
     ![选择节点](/images/创建server.png)

4. 选择节点，我选的是 Atlanta，通过测速网站看翻墙相对快。
     ![选择节点](/images/选择节点.png)

5. 选择配置，仅个人使用的话，选最便宜的就可以了。(CentOS选7x64, 8x64可能会报错"failed to install python")
     ![选择配置](/images/选择配置.png)

6. 点击deploy now， 创建成功后，可以看到当前创建的服务器状态为 Installing，等待几分钟后，就可以就变成 Running 了。
     ![创建server成功](/images/创建server成功.png)

7. 点进去，查看该服务器 IP，Username，Password。

# 连接远程服务器

## mac

mac 系统的话，可以直接用终端连接服务器：ssh root@149.28.90.255，再输入密码即可。
     ![mac连接服务器](/images/mac连接服务器.png)

## windows

windows 系统的话，可以直接用 SecureCRT 连接服务器或者使用Putty一个免费SSH客户端
     ![windows连接服务器](/images/windows连接服务器.jpeg)

ps：连接成功后，可以输入 passwd 来修改密码

# 搭建 Shadowsocks

1. 运行以下三条命令:

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh

chmod +x shadowsocks-all.sh

./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

2. 选择脚本（Python、R、Go、libev），任选一个：

```bash
Which Shadowsocks server you'd select:
1.Shadowsocks-Python
2.ShadowsocksR
3.Shadowsocks-Go
4.Shadowsocks-libev
Please enter a number (default 1):
```

我选择Shadowsocks-Go，输入3......然后，输入密码和端口（也就是你翻墙时所用的密码和端口）

3. 安装成功后，命令行出现：

Congratulations, Shadowsocks-Go server install completed!
Your Server IP        :  45.32.73.59
Your Server Port      :  8989
Your Password         :  teddysun.com
Your Encryption Method:  aes-256-cfb

Welcome to visit: https://teddysun.com/486.html
Enjoy it!

# Shadowsocks 客户端

下载对应系统的客户端并配置服务器

- [shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows/releases)
     ![windows_shadowsocks客户端](/images/windows_shadowsocks客户端.png)

- [shadowsocks-mac](https://github.com/shadowsocks/ShadowsocksX-NG/releases)
     ![mac_shadowsocks客户端](/images/mac_shadowsocks客户端.png)

- [shadowsocks-android](https://github.com/shadowsocks/shadowsocks-android/releases)
     ![android_shadowsocks客户端](/images/android_shadowsocks客户端.jpg)

# 使用 Chrome 插件 —— Proxy SwitchyOmega

方便在不同代理之间切换，例如直接连接或者自动代理模式。

去chrome商场安装 [SwitchyOmega插件](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?utm_source=chrome-ntp-icon)。

1. 设置 proxy模式

```xml
代理协议：socks5
代理服务器：127.0.0.1 // 本地ip
代理端口：1080 // Shadowsocks客户端配置的代理端口
```

![设置proxy模式](/images/设置proxy模式.png)

点击左边的“应用选项让配置生效”

2. auto switch

设置需要走代理的网站

![setProxyWeb](/images/setProxyWeb.png)

3. 切换模式

![切换模式](/images/切换模式.png)

若能正确访问[google](www.google.com)，即表示证明你搭建Shadowsocks成功了!

# 速度

YouTube 详情显示连接速度
视频右键，选择 stats for nerds

![youtube_connection_speed](/images/youtube_connection_speed.png)

# 全国 Ping 测试

本地 cmd 运行 ping ip 查看延迟和丢包。

![全国Ping测试](/images/全国Ping测试.png)

# IP 全球 Ping 测试

http://ping.pe/

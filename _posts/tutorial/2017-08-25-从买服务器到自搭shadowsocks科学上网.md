---
layout: post
title: 从买服务器到自搭shadowsocks科学上网 
category: tutorial
keywords: shadowsocks
---

# 1. 如何拥有一个shadowsocks访问节点
如果想要科学上网，你需要有一个境外的服务器来转发你的所有访问流量。那么这个境外的服务器如何而来呢？稳定长期的办法有两个：

- 从VPN服务商那里购买账号。
- 买个境外服务器，自己搭提子。

2016年我的主要翻墙手段是第一种，当时买的是一年期的“一枝红杏VPN”。他家价格还公道，一百多，最多三人同时使用，节点也不少。

不过今年（2017）我想拥有一个自己的服务器，一边充当翻墙梯子，一边也可以架一些网站上去。所以索性就从RamNode买了一个服务器。节点选的LA，系统选的ubuntu 14.04。
![Screen Shot 2017-08-25 at 10.17.21 am.png-56.6kB][1]

**［敲黑板］**重点来了，拥有了服务器，怎么开启服务呢？ubuntu系统几行命令就搞定啦。

```    
安装
Debian / Ubuntu:
$ apt-get install python-pip
$ pip install shadowsocks

使用
$ ssserver -p 443 -k password -m rc4-md5
如果要后台运行：
$ ssserver -p 443 -k password -m rc4-md5 --user nobody -d start
如果要停止：
$ ssserver -d stop
如果要检查日志：
$ less /var/log/shadowsocks.log
```
Ref: https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E


# 2. 本机配置

笔者的电脑是Mac，所以这里仅纪录Mac的配置方法。

### 2.1 下载ShadowsocksX
下载链接：https://osdn.net/projects/sfnet_shadowsocksgui/downloads/dist/ShadowsocksX-2.6.3.dmg/

[Windows] https://github.com/shadowsocks/shadowsocks-windows/releases/download/3.4.3/Shadowsocks-3.4.3.zip

### 2.2 配置ShadowsocksX
在ShadowsocksX的server preferences中配置server的IP，Port，密码，加密类型。

![Screen Shot 2017-08-25 at 10.33.02 am.png-61.5kB][2]

配置完成后，开启ShadowsocksX，你的浏览器就可以科学上google啦！

![Screen Shot 2017-08-25 at 10.36.41 am.png-134.3kB][3]

### 2.3 [码农专属] 终端中科学上网
作为一个码农，终端中wget之类的命令不能翻墙是在是痛。OS X 10.10亲测Proxychains可以帮忙实现。

```
1. 安装

安装 Proxychains ：使用 Terminal 运行以下命令
$ brew install proxychains-ng

查看 shadowsocks 本地端口号（端口号默认是 1080）。打开 ~/.ShadowsocksX/gfwlist.js 文件

2. 设置 Proxychains 安装目录下的 proxychains.conf 文件

打开 proxychains.conf 文件所在位置，在我电脑上为 /usr/local/etc/proxychains.conf

打开 proxychains.conf 文件，并在 [ProxyList] 下面添加 socks5 127.0.0.1 1080（如果 [ProxyList] 下面有其他的内容，则将其删掉或注释掉），1080 即为 shadowsocks 的本地端口号。

3. 使用
在要使用代理的命令前面加上 proxychains4，即可使其命令走代理途径。比如正常情况下运行 wget www.google.com，那么让其走代理途径则是 proxychains4 wget www.google.com，其他命令同理类推。

```
Ref: http://www.jianshu.com/p/bee7c63c3d50


  [1]: http://static.zybuluo.com/lrl940607/2yg8m7rremk8k44w9nvakvhx/Screen%20Shot%202017-08-25%20at%2010.17.21%20am.png
  [2]: http://static.zybuluo.com/lrl940607/ijtc35t5bi3mif1le3guxdpy/Screen%20Shot%202017-08-25%20at%2010.33.02%20am.png
  [3]: http://static.zybuluo.com/lrl940607/apnmukw5gy9hql1s4n4yxlhl/Screen%20Shot%202017-08-25%20at%2010.36.41%20am.png
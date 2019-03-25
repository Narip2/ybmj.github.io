---
title: 在VPS上配置SS
comments: true
date: 2018-12-25 22:30:48
categories:
- 工作环境配置
---

## 购买VPS

在Vultr这个网站上购买相应的VPS，我的选择是ubuntu18.04， 3.5美元每月。

<!--more-->
## SSH连接

购买完成后，会给你服务器的IP，用户名和密码。

对于**Linux**或**Mac**用户，可以直接在终端用ssh连接服务器

> ssh root@服务器的IP地址

之后输入服务器的密码就可以啦

---

对于**Windows**用户，可以下载Xshell，从而实现远程控制。

端口号默认是22 （可以改

## 配置SS

### 服务器端

远程连接的目的就是为了配置SS，方法有很多种。

这个方法出自：[链接](http://blog.51cto.com/13940125/2165848)

> git clone https://github.com/Flyzy2005/ss-fly

> ss-fly/ss-fly.sh -i 密码 端口

指定服务器端的密码和端口号

### 客户端

ubuntu下：[链接](https://blog.csdn.net/johinieli/article/details/79594954)

> sudo pip install shadowsocks

然后创建一个.json文件，里面放一些配置信息，这样就不用每次输命令了。

```json
{
    "server":"1.1.1.1",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"your passwd",
    "timeout":300,
    "method":"aes-256-cfb"
}
```

运行命令：
> sslocal -c /home/Desktop/ss.json

还可以设置开机启动：
```
# 打开图形化开机启动项管理界面
gnome-session-properties
# 添加(Add) -> 名称(name)和描述(comment)随便填，命令(Command)填写如下： 
sslocal -c /etc/ss.json
```

### 配置代理：

**全局模式：**

在 系统->网络 中设置手动代理，将ip地址和端口设置为127.0.0.1 和 之前设置的端口号。

**PAC模式：**

我们选择在本地生成.pac文件

> sudo pip install genpac

>genpac --proxy="SOCKS5 127.0.0.1:9696" --gfwlist-proxy="SOCKS5 127.0.0.1:9696" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
 
然后在 系统->网络 中设置自动代理，URL填写生成的.pac文件的路径即可


Windows 下：

下载相应的SS客户端即可。


IOS 下：

去app商店里，找一个可以进行ss连接的软件即可。

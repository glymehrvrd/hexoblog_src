---
title: 内网通过代理服务器并使用Shadowsocks联网
date: 2016-01-09 15:45:27
categories: technology
tags:
- WINDOWS
---

通过CCProxy可以便捷的搭建一个代理服务器，但是如何通过该代理服务器上的Shadowsocks连接网络呢？

<!-- more -->

这就需要用到端口转发（Port Forwarding）的功能，Windows7以上的系统已经自带简单的端口转发功能。  
```bash
netsh add v4tov4 listenaddress=IPADDR listenport=PORT connectaddress=127.0.0.1 connectport=1080
```
在CMD中输入该命令，就可以把PORT端口收到的数据转发到本机的1080端口上，1080端口正是代理服务器的Shadowsocks监听的端口。然后还需要把PORT加入防火墙的例外中。被代理的主机只需要通过Socks连接该端口就可以达到使用代理服务器的Shadowsocks科学上网的目的。
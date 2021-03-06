---
layout: blog
istop: false
title: "远程连接windows上的docker容器"
background-image: https://o243f9mnq.qnssl.com/2017/06/116099051.jpg
date:  2017-10-19
category: docker
tags:
- docker
- 路由
- 虚拟机
---


在Windows宿主机中连接虚拟机中的Docker容器
===
如果此时在宿主机中pingDocker容器是ping不同的，因为在宿主机上没有通往192.168.1.0/24网络的路由，宿主机会将发往192.168.1.0/24网络的数据发往默认路由，这样就无法到达容器。

虚拟机ip: 192.168.233.129
容器网段: 172.17.0.0

---
解决方法：
1. 首先要保证在虚拟机中能够连接到Docker容器中，用ping测试是否通畅

2. 关闭虚拟中的防火墙： systemctl stop firewalld.service

3. 打开宿主机（windows）的cmd,在其中添加通往172.17.0.0/16网络的路由。
4. 通往172.17.0.0/16网络的数据包由192.168.233.129来转发
> 可能需要管理员身份运行.

```
route add 172.17.0.0 mask 255.255.255.0 192.168.233.129
```
5. 验证效果: 会显示ipv4路由表
```
route print 172.17.0.0
```
6. ping 容器ip试试.
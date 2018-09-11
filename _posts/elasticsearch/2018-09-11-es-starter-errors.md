---
layout: blog
istop: false
title: "es遇到的错误"
date:  2018-09-11
category: elasticsearch
tags:
- elasticsearch
---

es安装使用过程遇到的错误
===

### java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in

Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true
```
# 在`elasticsearch.yml`在Memory下面添加:
bootstrap.system_call_filter: false 
```

### 无法远程访问
修改`config/elasticsearch.yml `:
```
network.host: <IP地址>
```
### 无法以root用户启动
```
# 新建一个用户
useradd es
# 修改es的文件夹所有者为es
chown es ./elasticsearch -R
```
### max number of threads [2048] for user [es] is too low, increase to at least [4096]
```
# 切换为root用户,修改
vi /etc/security/limits.d/90-nproc.conf 
# 增加一行(es为启动es的linux用户名)
es         soft    nproc     4096
```

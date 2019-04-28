---
layout: post
title: "Ubuntu通过SSH连接服务器"
tags: [Ubuntu, ssh]
comments: true
---

# 首先在服务器上安装ssh的服务器端
---
$ sudo aptitude install openssh-server

# 启动ssh-server。
---
$ /etc/init.d/ssh restart

# 确认ssh-server已经正常工作。
---
$ netstat -tlp

> tcp6 0 0 *:ssh *:* LISTEN -

看到上面这一行输出说明ssh-server已经在运行了。

# 在Ubuntu客户端通过ssh登录服务器。
---
假设服务器的IP地址是192.168.0.103，登录的用户名是xxx。

$ ssh -l xxx 192.168.0.103

接下来会提示输入密码，然后就能成功登录到服务器上了。
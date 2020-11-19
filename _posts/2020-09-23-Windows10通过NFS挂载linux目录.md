---
layout: post
title: "Windows10通过NFS挂载linux目录"
tags: [windows 10,nfs,linux]
comments: true
---

大致分为以下三大步骤：

一、启动NFS服务器

二、启动NFS客户端

三、挂载NFS目录

工具：

win10、虚拟机Ubuntu18.0系统

一 启动linux的NFS服务端：

以下均为Ubuntu操作系统命令: #sudo apt-get install nfs-kernel-server

选择你需要挂载的文件系统  #vim /etc/exports   (可以使用其他得编辑器)

编辑配置文件最后一行添加    /nfsroot  *(rw,sync,no_root_squash)  

然后保存

创建你刚刚配置添加得目录

mkdir /nfsroot

给目录授权

chmod -R 777 /nfsroot

chown -R nobody /nfsroot

重启NFS服务

#/etc/init.d/nfs-kernel-server restart

二 启动windos NFS客户端服务:

打开控制面板->程序->打开或关闭windows功能->NFS客户端

保存后记得重启电脑。

三 挂载NFS目录

windows下按下面得命令

win+R->cmd

mount IP:/nfsroot X:    (IP为NFS服务器得IP，/nfsroot为你挂载得目录路径)

成功挂载,打开我的点脑,你即可在你网络位置看到 X:盘了

window连接linux nfs服务器 —— 网络错误 53

NFS服务器有一个”在非安全模式工作（允许更高的端口号）“的选项。Windows NFS客户端经常使用的是大的端口号。你可以在你的共享项设置中开启这个选项 例如：/share *(insecure,rw)

或者

重启服务 sudo service nfs-kernel-server restart
---
layout: post
title: "Jetson nano 测CPU GPU温度"
tags: [linux, cpu, gpu, jetson nano, ubuntu]
comments: true
---

## 安装python
> sudo apt-get install python-pip

```
如果提示无法下载python-pip-whl_***.deb
需要更新软件包
sudo apt-get update
```

## 安装jetson-stats
> sudo -H pip install jetson-stats

## 更新
> sudo -H pip install -U jetson-stats

## 工具地址
> https://github.com/rbonghi/jetson_stats

```
如果遇到问题 AttributeError: module ‘pip.main’ has no attribute ‘_main’
解决方法：
sudo apt-get install python-pip python-dev build-essential #下载并安装pip
pip install --upgrade pip #更新pip

#打开pip的配置文件
#这里一定要记得加sudo，也就是以管理员身份打开，否则没有权限修改文件
sudo gedit /usr/bin/pip
#按照以下内容修改文件的对应部分
from pip import main
if name == ‘main’:
sys.exit(main._main())
```

## 使用
> sudo jtop


/usr/bin/tegrastats
/sys/devices/virtual/thermal/thermal_zone[0-5]]/type
/sys/devices/virtual/thermal/thermal_zone[0-5]]/temp

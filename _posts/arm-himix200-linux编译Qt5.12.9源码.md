---
layout: post
title: "arm-himix200-linux编译Qt5.12.9源码"
tags: [qt,himix200,configure,qmake.conf]
comments: true
---

# 安装arm-himix200-linux交叉编译器
1. tar -zxvf arm-himix200-linux.tgz
2. cd arm-himix200-linux
3. sudo ./arm-himix200-linux.install
```shell
Installing HuaWei LiteOS Linux at /opt/hisi-linux/x86-arm
mkdir: created directory '/opt/hisi-linux/x86-arm/arm-himix200-linux'
Extract cross tools ...
export path /opt/hisi-linux/x86-arm/arm-himix200-linux/bin
```
4. 检查/etc/profile是否写入环境变量
```shell
# HuaWei LiteOS Linux, Cross-Toolchain PATH
export PATH="/opt/hisi-linux/x86-arm/arm-himix200-linux/bin:$PATH"
#
```
5. source /etc/profile

* `注意每次shell重启之后需要执行source /etc/profile`

# 创建arm-himix200-linux的qmake.conf
备份源码编译环境
> cp -rf /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-gnueabi-g++ /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++
> vi /opt/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++/qmake.conf
```conf
#
# qmake configuration for building with arm-linux-gnueabi-g++
#

MAKEFILE_GENERATOR      = UNIX
CONFIG                 += incremental
QMAKE_INCREMENTAL_STYLE = sublib

include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)

# modifications to g++.conf
QMAKE_CC                = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-gcc
QMAKE_CXX               = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++
QMAKE_LINK              = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++
QMAKE_LINK_SHLIB        = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++

# modifications to linux.conf
QMAKE_AR                = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-ar cqs
QMAKE_OBJCOPY           = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-objcopy
QMAKE_STRIP             = /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-strip

load(qt_config)

# modifications to gcc-base.conf
QMAKE_CFLAGS_RELEASE += -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4
QMAKE_CXXFLAGS_RELEASE += -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive
```

# 配置QT源文件，支持OpenGL
QT根目录新建脚本文件configure.sh增加执行权限,配置如下：
```
#!/bin/bash
./configure \
-v \
-opensource \
-confirm-license \
-release \
-shared \
-xplatform arm-himix200-linux-g++ \
-prefix /opt/qt5.12.9_hi3559av100_release \
-opengl es2 \
-eglfs
-make libs \
-nomake examples \
-nomake tests \
-nomake tools \
-no-iconv \
-no-openssl \
-no-dbus \
-no-cups \
-no-pkg-config \
-skip qt3d \
-skip qtcanvas3d \
-skip qtdeclarative \
| tee ./qt_configure_information
```

# 编译debug版本且支持tslib的编译脚本
```
#!/bin/bash
./../qt-everywhere-src-5.12.9/configure \
-xplatform arm-himix200-linux-g++ \
-prefix /opt/qt5.12.9_hi3559av100_release_debug \
-v \
-opensource \
-confirm-license \
-debug \
-shared \
-recheck-all \
-make libs \
-tslib \
-nomake examples \
-nomake tests \
-nomake tools \
-no-opengl \
-no-openssl \
-no-dbus \
-no-iconv \
-skip qt3d \
-skip qtcanvas3d \
-skip qtdeclarative \
-I/home/wh/nfsdir/tslib3559/include -L/home/wh/nfsdir/tslib3559/lib | tee ./qt_configure_information
```

```
#!/bin/bash
./../qt-everywhere-src-5.12.9/configure \
-prefix /opt/qt5.12.9_hi3516dv300 \
-xplatform linux-arm-himix200-linux-g++ \
-v \
-opensource -confirm-license -release -silent -strip -no-rpath -no-pch \
-no-pkg-config \
-make libs -nomake examples -nomake tests -nomake tools \
-no-iconv -qt-zlib -qt-pcre \
-no-openssl \
-qt-freetype \
-no-opengl \
-linuxfb \
-qt-libpng \
-qt-libjpeg \
-qt-sqlite \
-no-assimp \
-skip qtserialbus \

make -j4

make install
```

# 编译
make -j$(nproc)
make install

# 问题
执行./configure报错 

`arm-himix200-linux: not found` 

是因为环境问题

查看编译器版本命令

`arm-himix200-linux -v` 

结果：
```
bash: /opt/hisi-linux/x86-arm/aarch64-himix100-linux--g++/target/bin/arm-himix200-linux: No such file or directory 
```
说明：gcc编译器找不到，但是编译器确实是安装完成了，输入arm，按两次Tab后也能找到交叉编译gcc文件，交叉编译器虽然安装了，但是交叉编译器的运行缺少库文件，这是因为宿主机是64位，而交叉编译器是针对32位的开发板制作的。所以要安装对应的32位库。我的linux宿主机是ubuntu 16.04 64位。

安装命令：
```
sudo apt install lib32z1-dev
```

安装完成后，再执行

`arm-himix200-linux -v`

即可出现gcc的版本信息，表示gcc安装成功。

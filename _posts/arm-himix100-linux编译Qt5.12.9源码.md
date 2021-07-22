---
layout: post
title: "aarch64-himix100-linux编译Qt5.12.9源码"
tags: [qt,himix100,configure,qmake.conf]
comments: true
---

# 编译海思3559sdk,最后在mpp/sample/gpu/release下会生成三个目录
```
inclue 待会交叉编译OpenGL会用到
ko 需要拷贝到开发板加载
lib 交叉编译和运行都需要
```

# 安装aarch64-himix100-linux交叉编译器
1. 解压缩aarch64-himix100-linux
2. 执行脚本aarch64-himix100-linux.install
3. 成功之后生成/opt/hisi-linux目录
4. 检查/etc/profile是否写入环境变量
5. 执行source /etc/profile

* `注意每次shell重启之后需要执行source /etc/profile`

# 配置交叉编译器软链接
1. 将target/usr 链接到根目录usr
> cd /opt/hisi-linux/x86-arm/aarch64-himix100-linux
> sudo ln -s ./target/usr usr
2. 将aarch64-linux-gnu链接到/usr/lib下
> sudo ln -s /opt/hisi-linux/x86-arm/aarch64-himix100-linux/aarch64-linux-gnu /usr/lib/aarch64-linux-gnu
* `拷贝GPU头文件和库,将海思SDK里mpp/component/gpu/release目录下的include目录和lib目录拷贝到opt/hisi-linux/x86-arm/aarch64-himix100-linux/usr目录下`

# 在qtbase/mkspecs/linux-aarch64-himix100-g++/qmake.conf文件加入对mali的支持
备份源码编译环境
> cp -rf ./qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-aarch64-gnu-g++ ./qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-aarch64-himix100-g++
```
# gpu  
QMAKE_LIBS_EGL         += -lmali
QMAKE_LIBS_OPENGL_ES2  += -lmali
DEFINES                += EGL_EGLEXT_PROTOTYPES GL_GLEXT_PROTOTYPES  EGL_FBDEV EGL_API_FBDEV EGL_API_MIDGARD PLATFORM_MALI700

# modifications to g++.conf
QMAKE_CC                = aarch64-himix100-linux-gcc
QMAKE_CXX               = aarch64-himix100-linux-g++
QMAKE_LINK              = aarch64-himix100-linux-g++
QMAKE_LINK_SHLIB        = aarch64-himix100-linux-g++

# modifications to linux.conf
QMAKE_AR                = aarch64-himix100-linux-ar cqs
QMAKE_OBJCOPY           = aarch64-himix100-linux-objcopy
QMAKE_NM                = aarch64-himix100-linux-nm -P
QMAKE_STRIP             = aarch64-himix100-linux-strip
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
-xplatform linux-aarch64-himix100-g++ \
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
-xplatform linux-aarch64-himix100-g++ \
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

# 编译
make -j$(nproc)
make install

# 问题
执行./configure报错 

`aarch64-himix100-linux-g++: not found` 

是因为环境问题

查看编译器版本命令

`aarch64-himix100-linux-g++ -v` 

结果：
```
bash: /opt/hisi-linux/x86-arm/aarch64-himix100-linux--g++/target/bin/aarch64-himix100-linux-g++: No such file or directory 
```
说明：gcc编译器找不到，但是编译器确实是安装完成了，输入arm，按两次Tab后也能找到交叉编译gcc文件，交叉编译器虽然安装了，但是交叉编译器的运行缺少库文件，这是因为宿主机是64位，而交叉编译器是针对32位的开发板制作的。所以要安装对应的32位库。我的linux宿主机是ubuntu 16.04 64位。

安装命令：
```
sudo apt install lib32z1-dev
```

安装完成后，再执行

`aarch64-himix100-linux-g++ -v`

即可出现gcc的版本信息，表示gcc安装成功。

# aarch64-himix100-linux-gcc 6.3.0(这个版本有点小问题，使用前需要先清除本地化设置)
> export LANG=C
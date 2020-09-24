---
layout: post
title: "QtCreator加入cmake"
tags: [cmake,qtcreator]
comments: true
---

# QtCreator加入cmake
1. 下载CMake并解压 
下载地址:https://cmake.org/download/
```
tar xvf cmake-3.12.0-rc3.tar.gz
```
2. 编译并安装CMake
```
$ cd cmake-3.12.0-rc3/
$ ./bootstrap
$ make -j8
$ sudo make install
```
```
编译错误
Could NOT find OpenSSL, try to set the path to OpenSSL root folder in the system variable OPENSSL_ROOT_DIR (missing: OPENSSL_LIBRARIES OPENSSL_INCLUDE_DIR) 
在 Ubuntu 系统上的解决方法是，在命令行输入如下命令：sudo apt-get install libssl-dev
```
3. 配置Qt
```
Qt-->Tools-->Options-->Build & Run-->CMake-->Add
Name：随意，暂取cmake-3.12.0-rc3
Path：/home/yong/Downloads/cmake-3.12.0-rc3/bin/cmake，即编译出来的CMake命令的绝对路径。
Qt->Tools->Options->Build & Run->Kits->Desktop(dafault)->CMake Tool 选择 cmake-3.12.0-rc3
```

# 编译cmake问题
```
Could NOT find OpenSSL, try to set the path to OpenSSL root folder in the system variable OPENSSL_ROOT_DIR (missing: OPENSSL_LIBRARIES OPENSSL_INCLUDE_DIR) 
在 Ubuntu 系统上的解决方法是，在命令行输入如下命令：sudo apt-get install libssl-dev
```

# 虚拟机中qtcreator通过cmake编译代码报错
```
error: undefined reference to `__cxa_begin_catch'
```
解决方法:CMakelists.text里面target_link_libraries加入libstdc++.so
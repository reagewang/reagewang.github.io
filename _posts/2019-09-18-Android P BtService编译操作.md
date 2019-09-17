---
layout: post
title: "Android P BtService编译操作"
tags: [bluetooth,btservice,android p,build]
comments: true
---

编译之前设置环境
> source build/envsetup.sh

> lunch car_mt2712-userdebug

进入BtService目录,执行
> mm

生成文件目录
> out/target/product/car_mt2712/system/priv-app/BtService/BtService.apk

编译成功结果
> #### build completed successfully (01:31 (mm:ss)) ####


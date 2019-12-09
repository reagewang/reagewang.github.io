---
layout: post
title: "Android mk写法讲解"
tags: [android,mk,apk]
comments: true
---

# LOCAL_PATH
> 每个 Android.mk 文件必须以定义 LOCAL_PATH 为开始，它用于在开发 tree 中查找源文件
>
> LOCAL_PATH := $(call my-dir)

# include $(CLEAR_VARS)
> CLEAR_VARS 变量由 Build System 提供，并指向一个指定的 GNU Makefile，由它负责清理很多 LOCAL_xxx。例如：LOCAL_MODULE, LOCAL_SRC_FILES, LOCAL_STATIC_LIBRARIES 等，但`不清理 LOCAL_PATH`。

# LOCAL_MODULE_TAGS 
> 可选定义，表示在什么版本情况下编译该版本，默认 `optional`

||说明|
|-|-|
|user|指该模块只在 user 版本下才编译|
|eng|指该模块只在 eng 版本下才编译|
|tests|指该模块只在 tests 版本下才编译|
|optional|指该模块在所有版本下都编译|

# LOCAL_MODULE
> 模块名，可不用定义，默认 = $(LOCAL_PACKAGE_NAME)，不能和既有模块相同，如果该变量未设置，则使用LOCAL_PACKAGE_NAME，如果再没有，就会编译失败。

# LOCAL_PACKAGE_NAME

# LOCAL_CERTIFICATE
> 在什么情况下签名
>
> LOCAL_CERTIFICATE := platform

||说明|
|-|-|
|testkey|普通 APK，默认情况下使用|
|platform|该 APK 完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的 APK 所在进程的UID 为 system，可以参见 Settings|
|shared|该 APK 需要和 home/contacts 进程共享数据，可以参见 Launcher|
|media|该 APK 是 media/download 系统中的一环，可以参见 Gallery|

# LOCAL_SRC_FILES
> 指定 src 目录 
> 
> LOCAL_SRC_FILES := $(call all-java-files-under,java)

# LOCAL_MODULE_CLASS
> 指定模块的类型，可不用定义

||说明|
|-|-|
|APPS|编译 apk 文件|
|JAVA_LIBRAYIES|编译 jar 包|
|SHARED_LIBRAYIES|定义动态库文件|
|EXECUTABLES|编译可执行文件|

# include $(BUILD_PACKAGE)
> 表示生成一个 apk，它可以是多种类型

||说明|
|-|-|
|BUILD_PACKAGE|既可以编apk，也可以编资源包文件，但是需要指定LOCAL_EXPORT_PACKAGE_RESOURCES:=true|
|BUILD_JAVA_LIBRARY|java共享库|
|BUILD_STATIC_JAVA_LIBRARY|java静态库|
|BUILD_EXECUTABLE|执行文件|
|BUILD_SHARED_LIBRARY|native共享库|
|BUILD_STATIC_LIBRARY|native静态库|

# LOCAL_PRIVATE_PLATFORM_APIS
> 设置后，会使用 sdk 的 hide 的 api 来编译。
> 
> LOCAL_PRIVATE_PLATFORM_APIS := true

# LOCAL_SDK_VERSION 
> 这个编译配置，就会使编译的应用不能访问 hide 的 api，有时一些系统的 class 被 import 后编译时说找不到这个类，就是这个原因造成的。

# LOCAL_USE_AAPT2
> 引入系统资源文件
>
> LOCAL_USE_AAPT2 := true

# LOCAL_STATIC_ANDROID_LIBRARIES
> 导入系统依赖
> 
> LOCAL_STATIC_ANDROID_LIBRARIES := \
android-support-design \
android-support-v4 \
android-support-v7-appcompat \
android-support-v7-recyclerview 

# LOCAL_RESOURCE_DIR
> 资源文件，可选定义，推荐不定义 
> 
> LOCAL_RESOURCE_DIR = $(LOCAL_PATH)/res \
frameworks/support/v7/appcompat/res \
frameworks/support/design/res

# LOCAL_PRIVILEGED_MODULE
> 将 APK 预置到 system/priv-app 里
> 
> LOCAL_PRIVILEGED_MODULE := true

# LOCAL_DEX_PREOPT
> 关闭 dex 优化来提高调试过程，把编译后的 APK 直接替换安装 adb install -r XXX.apk，不然 APK 得 Push 到 system/app，重启设备。
> 
> LOCAL_DEX_PREOPT := false

# LOCAL_PROGUARD_ENABLED
> 制定编译的工程，不要使用代码混淆的工具进行代码混淆
>
> LOCAL_PROGUARD_ENABLED := disabled

# 引用第三方 jar 包
```
声明我们 jar 包所在的目录,这行代码的意思大概可以理解成这样，声明一个变量 AndroidUtil，它的 value 是 libs/AndroidUtil.jar

LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := AndroidUtil:libs/AndroidUtil.jar

引用我们声明 jar 包的变量

include $(CLEAR_VARS)
# 省略其他
LOCAL_STATIC_JAVA_LIBRARIES := \
        AndroidUtil
# 省略其他
include $(BUILD_PACKAGE)
``` 

# 引用 so 库
> 假设，我们当前目录下的 libs/armeabi 有 libBaiduMapSDK1.so，我们想引用它，有两种方法，可以在根目录 Android.mk 引用 so 库，也可以在 libs 下再建个 Android.mk 配置好 so 库，然后 include，推荐第二种方式。
```
libs/Android.mk

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := .so
LOCAL_MODULE := libBaiduMapSDK1
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_SRC_FILES :=libs/armeabi/$(LOCAL_MODULE).so
LOCAL_MULTILIB := both
include $(BUILD_PREBUILT)
```
```
include $(CLEAR_VARS)
# 省略其他
LOCAL_JNI_SHARED_LIBRARIES :=  \
        libBaiduMapSDK1 
# 省略其他
include $(BUILD_PACKAGE)

# 引用第三方 so 库
include $(LOCAL_PATH)/libs/Android.mk
```

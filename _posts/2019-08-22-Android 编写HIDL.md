---
layout: post
title: "Android 编写HIDL"
tags: [android,framework,hidl,hal]
comments: true
---
  
  
# AOSP有哪些HAL：

![](https://upload-images.jianshu.io/upload_images/5439660-8164e88cfd918ab4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/608/format/webp)

这些啊都是Android设备上的硬件，因为Google理论上只关心Android的框架层和上层软件，但是上层软件依赖于底层的硬件实现，但是每家手机厂商，或者说是CPU厂商底层硬件的实现都是不一样的，所以这个HAL层基本都是手机厂商或者CPU厂商去实现的。

Google只是作为一个框架的指导，和Framework层API的接口定义，这些接口的实现都得由HAL去完成。

## 示例 HelloWorld
### HIDL 接口文件定义
进入代码，我们假设HelloWorld作为标准AOSP的HAL，我们就把代码揉进标准HAL层去，进入代码目录创建HIDL目录：
```
mkdir -p hardware/interfaces/helloworld/1.0/default
```
接着创建接口描述文件IHelloWorld.hal，放在刚才创建的目录中
```
package android.hardware.helloworld@1.0;

interface IHelloWorld {
    funHelloWorld(string name) generates (string result);
};
```
这是一个Google定义的语言格式，C++和Java的结合体。

这里我们定义了一个IHelloWorld接口文件，简单的添加了一个funHelloWorld接口，传入是一个string，返回一个string，后面我们会来实现这个接口。

### 生成 HAL 相关文件
Google帮我们提供了一些工具来生成HAL层相关的代码框架和代码实例，这样子我们只需要关心实现部分，而不需要写一堆无用代码，浪费时间在搞Makefile和一些低级错误上。

使用hidl-gen工具
```
# PACKAGE=android.hardware.helloworld@1.0
# LOC=hardware/interfaces/helloworld/1.0/default/
# make hidl-gen -j64
# hidl-gen -o $LOC -Lc++-impl -randroid.hardware:hardware/interfaces -randroid.hidl:system/libhidl/transport  $PACKAGE
# hidl-gen -o $LOC -Landroidbp-impl -randroid.hardware:hardware/interfaces -randroid.hidl:system/libhidl/transport $PACKAGE
```
然后使用脚本来更新Makefile，自动生成Android,mk, Android.bp
```
# ./hardware/interfaces/update-makefiles.sh
```
现在，我们来添加两个空文件：
```
touch hardware/interfaces/helloworld/1.0/default/android.hardware.helloworld@1.0-service.rc
touch hardware/interfaces/helloworld/1.0/default/service.cpp
```
现在我们的代码目录: hardware/interface/helloworld:
```
├── 1.0
│   ├── Android.bp
│   ├── Android.mk
│   ├── default
│   │   ├── Android.bp
│   │   ├── android.hardware.helloworld@1.0-service.rc
│   │   ├── HelloWorld.cpp
│   │   ├── HelloWorld.h
│   │   └── service.cpp
│   └── IHelloWorld.hal
└── Android.bp
```
我们写代码就写了一个IHelloWorld.hal，其余代码都是自动生成的，特别是HelloWorld.cpp和HelloWorld.h这两个文件是实现接口的关键文件。

### 实现 HAL 实现端的共享库
```c++
// HelloWorld.h
struct HelloWorld : public IHelloWorld {
    // Methods from IHelloWorld follow.
    Return<void> funHelloWorld(const hidl_string& name, funHelloWorld_cb _hidl_cb) override;
    // Methods from ::android::hidl::base::V1_0::IBase follow.
};

// FIXME: most likely delete, this is only for passthrough implementations
// extern "C" IHelloWorld* HIDL_FETCH_IHelloWorld(const char* name);
```
我们知道，HIDL的实现有两种方式，一种是Binderized模式，另一种是Passthrough模式，我们看到上面有两行注释掉的代码，看来这个代码是关键，来选择实现方式是Binderized还是Passthrough。

我们这里使用Passthrough模式来演示，其实大家后面尝试这两种方式后会发现其实这两种本质是一样的，目前大部分厂商使用的都是Passthrough来延续以前的很多代码，但是慢慢的都会被改掉的，所以我们来打开这个注释。
```c++
// HelloWorld.h
# ifndef ANDROID_HARDWARE_HELLOWORLD_V1_0_HELLOWORLD_H
# define ANDROID_HARDWARE_HELLOWORLD_V1_0_HELLOWORLD_H
# include <android/hardware/helloworld/1.0/IHelloWorld.h>
# include <hidl/MQDescriptor.h>
# include <hidl/Status.h>

namespace android {
namespace hardware {
namespace helloworld {
namespace V1_0 {
namespace implementation {

using ::android::hardware::hidl_array;
using ::android::hardware::hidl_memory;
using ::android::hardware::hidl_string;
using ::android::hardware::hidl_vec;
using ::android::hardware::Return;
using ::android::hardware::Void;
using ::android::sp;

struct HelloWorld : public IHelloWorld {
    // Methods from IHelloWorld follow.
    Return<void> funHelloWorld(const hidl_string& name, funHelloWorld_cb _hidl_cb) override;
    // Methods from ::android::hidl::base::V1_0::IBase follow.
};

// FIXME: most likely delete, this is only for passthrough implementations
extern "C" IHelloWorld* HIDL_FETCH_IHelloWorld(const char* name);

}  // namespace implementation
}  // namespace V1_0
}  // namespace helloworld
}  // namespace hardware
}  // namespace android

# endif  // ANDROID_HARDWARE_HELLOWORLD_V1_0_HELLOWORLD_H
```
```c++
// HelloWorld.cpp
# include "HelloWorld.h"

namespace android {
namespace hardware {
namespace helloworld {
namespace V1_0 {
namespace implementation {

// Methods from IHelloWorld follow.
Return<void> HelloWorld::funHelloWorld(const hidl_string& name, funHelloWorld_cb _hidl_cb) {
    // TODO implement
    char buf[100];
    ::memset(buf, 0x00, 100);
    ::snprintf(buf, 100, "Hello World, %s", name.c_str());
    hidl_string result(buf);

    _hidl_cb(result);
    return Void();
}

// Methods from ::android::hidl::base::V1_0::IBase follow.

IHelloWorld* HIDL_FETCH_IHelloWorld(const char* /* name */) {
    return new HelloWorld();
}

}  // namespace implementation
}  // namespace V1_0
}  // namespace helloworld
}  // namespace hardware
}  // namespace android
```
1. 我们打开了HIDL_FETCH的注释，让我们的HIDL使用Passthrough方式去实现
2. 添加funHelloWorld函数的实现，简单的做了字符串拼接（学过C/C++）的同学应该都看得懂

然后可以查看一下Android.bp文件看一下编译生成个啥
```
cc_library_shared {
    name: "android.hardware.helloworld@1.0-impl",
    relative_install_path: "hw",
    proprietary: true,
    srcs: [
        "HelloWorld.cpp",
    ],
    shared_libs: [
        "libhidlbase",
        "libhidltransport",
        "libutils",
        "android.hardware.helloworld@1.0",
    ],
}
```
最终会生成一个android.hardware.helloworld@1.0-impl.so, 生成在/vendor/lib64/hw/下，我们用mmm编译生成看看
```
$ mmm hardware/interfaces/helloworld/1.0/default/

PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=8.1.0
TARGET_PRODUCT=hon660
TARGET_BUILD_VARIANT=userdebug
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm64
TARGET_ARCH_VARIANT=armv8-a
TARGET_CPU_VARIANT=generic
TARGET_2ND_ARCH=arm
TARGET_2ND_ARCH_VARIANT=armv7-a-neon
TARGET_2ND_CPU_VARIANT=cortex-a53
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-3.16.0-48-generic-x86_64-with-Ubuntu-14.04-trusty
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=OPM1.171019.011

# OUT_DIR=out

[2/2] bootstrap out/soong/.minibootstrap/build.ninja.in
[1/1] out/soong/.bootstrap/bin/minibp out/soong/.bootstrap/build.ninja
[2/3] glob hardware/interfaces/*/Android.bp
[1/1] out/soong/.bootstrap/bin/soong_build out/soong/build.ninja
No need to regenerate ninja file
[100% 3/3] out/soong/.bootstrap/bin/soong_build out/soong/build.ninja
[100% 18/18] build 'out/target/product/hon660/obj/SHARED_LIBRARIES/android.hardware.helloworld@1.0-impl_intermediates/android.hardware.helloworld@1.0-impl.so.toc'

#### build completed successfully (02:35 (mm:ss))
```

### 调用流程
上面呢我们完成了实现端的代码和编译，我们这节来看一下整个HIDL的调用流程，因为里面涉及到好几个库，有好多同学都被这些库给搞混了，我们来看看这些库的顺序吧。

HIDL软件包中自动生成的文件会链接到与软件包同名的单个共享库。该共享库还会导出单个头文件IHelloWorld.h，用于在binder客户端和服务端的接口文件，下面的图诠释了我们的IHelloWorld.hal编译后生成的文件走向，从官网拷贝过来的，大家不要在乎文件名哈：

![](https://upload-images.jianshu.io/upload_images/5439660-385176b88e488239.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/576/format/webp)

* **IFoo.h** - 描述 C++ 类中的纯 IFoo 接口；它包含 IFoo.hal 文件中的 IFoo 接口中所定义的方法和类型，必要时会转换为 C++ 类型。不包含与用于实现此接口的 RPC 机制（例如 HwBinder）相关的详细信息。类的命名空间包含软件包名称和版本号，例如 ::android::hardware::samples::IFoo::V1_0。客户端和服务器都包含此标头：客户端用它来调用方法，服务器用它来实现这些方法。
* **IHwFoo.h** - 头文件，其中包含用于对接口中使用的数据类型进行序列化的函数的声明。开发者不得直接包含其标头（它不包含任何类）。
* **BpFoo.h** - 从 IFoo 继承的类，可描述接口的 HwBinder 代理（客户端）实现。开发者不得直接引用此类。
* **BnFoo.h** - 保存对 IFoo 实现的引用的类，可描述接口的 HwBinder 存根（服务器端）实现。开发者不得直接引用此类。
* **FooAll.cpp** - 包含 HwBinder 代理和 HwBinder 存根的实现的类。当客户端调用接口方法时，代理会自动从客户端封送参数，并将事务发送到绑定内核驱动程序，该内核驱动程序会将事务传送到另一端的存根（该存根随后会调用实际的服务器实现）。

这些文件的结构类似于由 `aidl-cpp` 生成的文件。独立于 HIDL 使用的 RPC 机制的唯一一个自动生成的文件是 IFoo.h，其他所有文件都与 HIDL 使用的 HwBinder RPC 机制相关联。因此，客户端和服务器实现不得直接引用除 IFoo 之外的任何内容。为了满足这项要求，请只包含 IFoo.h 并链接到生成的共享库。

我们这个实例会用到以下几个模块：
* android.hardware.helloworld@1.0-impl.so: HelloWorld模块实现端的代码编译生成，binder server端
* android.hardware.helloworld@1.0.so: HelloWorld模块调用端的代码，binder client端
* helloworld_hal_service: 通过直通式注册binder service，暴露接口给client调用
* android.hardware.helloworld@1.0-service.rc: Android native 进程入口

![](https://upload-images.jianshu.io/upload_images/5439660-bf82ce77f6ab243e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/808/format/webp)

### 启动binder server端进程
还记得我们之前创建的两个文件吗，我们还没有去实现呢，先来看一下rc文件
```
service helloworld_hal_service /vendor/bin/hw/android.hardware.helloworld@1.0-service
    class hal
    user system
    group system
```
很简单，就是在设备启动的时候执行/vendor/bin/hw/android.hardware.helloworld@1.0-service程序：
```
# define LOG_TAG "android.hardware.helloworld@1.0-service"
# include <android/hardware/helloworld/1.0/IHelloWorld.h>
# include <hidl/LegacySupport.h>

using android::hardware::helloworld::V1_0::IHelloWorld;
using android::hardware::defaultPassthroughServiceImplementation;

int main() {
    return defaultPassthroughServiceImplementation<IHelloWorld>();
}
```
这个service是注册了IHelloWorld接口文件里面的接口，作为binder server端，很简单就一句话，因为我们使用了passthrough的模式，Android帮我们封装了这个函数，不需要我们自己去addService啦。
```
cc_binary {
    name: "android.hardware.helloworld@1.0-service",
    defaults: ["hidl_defaults"],
    proprietary: true,
    relative_install_path: "hw",
    srcs: ["service.cpp"],
    init_rc: ["android.hardware.helloworld@1.0-service.rc"],
    shared_libs: [
        "libhidlbase",
        "libhidltransport",
        "libutils",
        "liblog",
        "android.hardware.helloworld@1.0",
        "android.hardware.helloworld@1.0-impl",
    ],
}
```
编译后可以在, vendor/bin/hw/下找到对应的文件。OK，我们server端的进程和实现端共享库已经完成了。

但是这个时候你如果烧录镜像，会发现这个进程会启动失败，原因是因为我们没有给这个进程配sepolicy，所以正确的做法是要给他加上selinux的权限，我们这里就不去做了，因为我们可以用root权限去手动起这个service。

好了，接下来要看看client的代码怎么写了。

### HIDL Client测试代码
我写代码喜欢一步一步来，每一步都搞个测试代码来测试，一来是验证每一步的功能，是为后面测试使用。

写代码不是一件难事，写好代码是一件不容易的事情，好的代码都是通过大量测试来改善的，没有谁可以一次性的写好代码，所以大家在设计阶段一定要把测试接口留出来，不然的话后面返工去re-design的。

好了，贴上我们的测试代码：
```c++
# include <android/hardware/helloworld/1.0/IHelloWorld.h>
# include <hidl/Status.h>
# include <hidl/LegacySupport.h>
# include <utils/misc.h>
# include <hidl/HidlSupport.h>
# include <stdio.h>

using android::hardware::helloworld::V1_0::IHelloWorld;
using android::sp;
using android::hardware::hidl_string;

int main()
{
    int ret;

    android::sp<IHelloWorld> service = IHelloWorld::getService();
    if(service == nullptr) {
        printf("Failed to get service\n");
        return -1;
    }

    service->funHelloWorld("JayZhang", [&](hidl_string result) {
                printf("%s\n", result.c_str());
        });

    return 0;
}
```
代码是相当的简单啊，实例化binder service，通过IHelloWorld::getService()，获取到binder server端接接口代理类，然后就可以调用他的方法了，我们这里调用funHelloWorld接口，然后通过callback获取结果。

贴上Makefile:
```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_PROPRIETARY_MODULE := true
LOCAL_MODULE := helloworld_test
LOCAL_SRC_FILES := \
    client.cpp \

LOCAL_SHARED_LIBRARIES := \
   liblog \
   libhidlbase \
   libutils \
   android.hardware.helloworld@1.0 \

include $(BUILD_EXECUTABLE)
```
记得在manifest文件里添加vendor接口的定义，不然在client端是没法拿到service的，在相应的manifest.xml里面加入：
```xml
<hal format="hidl">
    <name>android.hardware.helloworld</name>
    <transport>hwbinder</transport>
    <version>1.0</version>
    <interface>
        <name>IHelloWorld</name>
        <instance>default</instance>
    </interface>
</hal>
```
然后我们来测试一下代码,手动运行service：
```
WH:/ # /vendor/bin/hw/android.hardware.helloworld@1.0-service
```
运行测试代码：
```
WH:/ # helloworld_test
Hello World, JayZhang
WH:/ # 
```
看到没有，我们的测试代码传入"JayZhang"字符串，结果输出"Hello World, JayZhang", 符合我们的预期结果。


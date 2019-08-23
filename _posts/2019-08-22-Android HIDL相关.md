---
layout: post
title: "Android HIDL"
tags: [android,hidl,hal]
comments: true
---

# Android架构图

![](https://upload-images.jianshu.io/upload_images/7342824-9691627e4d4a6b4e?imageMogr2/auto-orient/strip%7CimageView2/2/w/486/format/webp)

# Project Treble
Treble 是 Google Android 团队的一项重大项目，意在 Android 操作系统框架在架构方面的一项重大改变，旨在让制造商以更低的成本更轻松、更快速地将设备更新到新版 Android 系统。

Android 7.x 及更早版本中没有正式的供应商接口，因此设备制造商必须更新大量 Android 代码才能将设备更新到新版 Android 系统。
## Treble 推出前的 Android 更新环境
![](https://upload-images.jianshu.io/upload_images/7342824-38cdf340754cecd6?imageMogr2/auto-orient/strip%7CimageView2/2/w/430/format/webp)

Treble 提供了一个稳定的新供应商接口，供设备制造商访问 Android 代码中特定于硬件的部分，这样一来，设备制造商只需更新 Android 操作系统框架，即可跳过芯片制造商直接提供新的 Android 版本。

## Treble 推出后的 Android 更新环境

![](https://upload-images.jianshu.io/upload_images/7342824-d659fee16a850470?imageMogr2/auto-orient/strip%7CimageView2/2/w/517/format/webp)

通俗的说是这样，在以往Android更新操作系统，供应商（照相、传感器...）或芯片制造商（高通、MTK...）必须要先进行硬件适配，然后交由制造商（手机品牌）生产手机。这个过程往往比较漫长，也就是为什么 Android 更新版本后，绝大多数手机（Google亲儿子除外）需要很久才能收到系统更新的通知。Treble 就是意在缩短这个过程，HIDL 是基本的技术支撑。

# HIDL介绍
HAL接口定义语言（简称HIDL）适用于指定HAL和其用户之间的接口的一种接口描述语言（IDL）。HIDL允许指定类型和方法调用。从更更烦的意义上来说HIDL适用于在独立编程的代码库之间通信的系统。

HIDL旨在用于进程间通信（IPC）。进程之间的通信经过Binder化。对于必须与进程相关联的代码库，还可以使用直通模式。

HIDL可指定数据结构和方法签名，这些内容会整理归类到接口中，而接口会汇集到软件包中。尽管HIDL具有一系列不同的关键字，C\+\+和JAVA程序员对HIDL的语法并不陌生。此外，HIDL还是用JAVA样式和注释。

## 碎片化问题

![](https://upload-images.jianshu.io/upload_images/5439660-709053bed709dab2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

由于Android的发展比较迅猛，各大手机厂商和芯片厂商都在做，Google当然作为一个领导者在指引我们做出更好的手机操作系统，但是由于版本太多，Android版本的碎片化越来越严重，而且系统的更新又是一个耗时和复杂的过程，Google试图来解决这个问题而引入了Treble。

比如：小米，华为，VIVO等厂商，他们维护自己的BSP，基本上他们的BSP包含几部分：

![](https://upload-images.jianshu.io/upload_images/5439660-bbc26fe9a9d7ab6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/601/format/webp)

**注意，这里的Framework是vendor修改过的，这样子的话，这四部分都耦合在一起，因为Google每次更新Android大版本，基本上都是framework的升级，与vendor改的代码理论上是可以独立开来的，所以Google尝试通过Treble来独立更新system.img来帮助vendor更快的移植新的Android版本。**

以前HAL是以so的形式存在的，作为一堆标准接口，供Android framework调用，无论是通过jni还是别的途径。

如果要被framework调用，那这些so就一定要存在于system分区，但是我们现在要把system分区独立开来，这样子，vendor修改的代码全部要在vendor分区，所以，引入了HIDL来解决这个问题。

vendor设计的HAL都以独立的service存在，每一个HAL模块都是一个独立的binder server进程，Android framework想用调用HAL的接口就必须作为binder的client来调用。

![](https://upload-images.jianshu.io/upload_images/5439660-ab420eff2bc2d164.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

## HAL类型
为了更好地实现模块化，Android 8.0 对 Android 操作系统底层进行了重新架构。作为此变化的一部分，运行 Android 8.0 的设备必须支持绑定式或直通式 HAL：

* 绑定式 HAL。以 HAL 接口定义语言 (HIDL) 表示的 HAL。这些 HAL 取代了早期 Android 版本中使用的传统 HAL 和旧版 HAL。在绑定式 HAL 中，Android 框架和 HAL 之间通过 Binder 进程间通信 (IPC) 调用进行通信。所有在推出时即搭载了 Android 8.0 或后续版本的设备都必须只支持绑定式 HAL。
* 直通式 HAL。以 HIDL 封装的传统 HAL 或旧版 HAL。这些 HAL 封装了现有的 HAL，可在绑定模式和 Same-Process（直通）模式下使用。升级到 Android 8.0 的设备可以使用直通式 HAL。

所谓绑定式，仅通过 Binder 化进行传输，这个在 Android AIDL 中已经应用了很久，提供了代码库独立的基本条件。

既然是进程间通信，难免会有耗时过程的问题，这对于速度和性能要求特别高地的进程来说，这是不利的，因此可以通过直通式 HAL 来弥补绑定式 HAL 的缺点。

必须使用直通式 HAL 的进程情况：

* android.hardware.graphics.mapper@1.0 将内存映射到其所属的进程中。
* android.hardware.renderscript@1.0 在同一进程中传递项(等同于 openGL)。

## HIDL 的演化历程

![](https://upload-images.jianshu.io/upload_images/7342824-00a1d4097c6d11cb?imageMogr2/auto-orient/strip%7CimageView2/2/w/596/format/webp)

## HIDL的设计目标
框架可以在无需重新构建 HAL 的情况下进行替换。HAL 将由供应商或 SOC 制造商构建，放置在设备的 /vendor 分区中，这样一来，框架就可以在其自己的分区中通过 OTA 进行替换，而无需重新编译 HAL。

## 软件包
已发布的 HIDL package包位于 Android 代码库的hardware/interfaces/或vendor/`$(vendorName)`目录下。

使用 HDIL 描述的 HAL 接口存放在这些目录下的.hal文件中。

|软件包前缀|对应位置|
|--|--|
| android.hardware.* |hardware/interfaces/* |
| android.frameworks.* | frameworks/hardware/interfaces/* |
| android.system.* | system/hardware/interfaces/* |
| android.hidl.* | system/libhidl/transport/* |

软件包目录中包含扩展名为 .hal 的文件。每个文件均必须包含一个指定文件所属的软件包和版本的 package 语句。文件 types.hal（如果存在）并不定义接口，而是定义软件包中每个接口可以访问的数据类型。

## HIDL文件的组织结构
每个 HIDL package包里都含有一个名为types.hal的文件，该文件中定义了这个包里所有 interface 共享的用户自定义数据类型，并且一般也会导入需要用到的其它包里的数据类型。 

当前包中新的定义的 interface 可以继承自从其它包里导入的 interface，这样的继承关系可以使用extend关键字实现。比如下面示例中的 1.1 版本包中的 IQuux 接口就继承自 1.0 版本包中的 IQuux 接口：
```
// types.hal
package android.hardware.example@1.1
import android.hardware.example@1.0  // 导入1.0的包
```
```
// IQuux.hal
package android.hardware.example@1.1
interface IQuux extends @1.0::IQuux {  // 继承1.0包中的接口
　　fromBarToFoo(foo.bar b) generates (foo f);  // 直接使用fromBarToFoo方法而不再在当前包中声明
}
```
由 Google 提供的包叫做`core package`，包名始终以`android.hardware.`开头，以子系统名加以区分。比如 NFC 包的名字就应该为android.hardware.nfc，摄像头包的名字就应该为android.hardware.camera。这些core包存放于`hardware/interfaces/`目录下。由各芯片厂商和ODM厂商提供的包叫做`non-core package`，包名形式一般以`vendor.$(vendorName).hardware.`开头，比如vendor.samsung.hardware.。这些non-core包一般存放于`vendor/$(vendorName)/interfaces/`目录下。 

包的版本使用主、次版本号进行描述，紧随包名之后。比如android.hardware.audio@2.0表述这个 audio 包的版本是 2.0，主版本号是 2，次版本号是 0。

此外，每个 HIDL 包在被发布后就不能再对其内容进行变动了，如果要增加或修改这个包里的接口或数据类型，应该新建一个新版本的包，在这个新版本的包里进行变更。

## 接口定义
除了 types.hal 之外，其他 .hal 文件均定义一个接口。接口通常定义如下：
```
// 一个完整的.hal文件
package android.hardware.audio@2.0;  // 当前package包名
import android.hardware.audio.common@2.0;  // 导入其它package包
import IDevice;  // 导入其它.hal
interface IDevicesFactory {  // 定义一个interface
　　typedef android.hardware.audio@2.0::Result Result;
　　enum Device : int32_t {  // 定义数据类型
　　　　PRIMARY,
　　　　A2DP,
　　　　USB,
　　　　R_SUBMIX,
　　　　STUB
　　};
　　/**
　　　* Opens an audio device. To close the device, it is necessary to release
　　　* references to the returned device object.
　　　*
　　　* @param device device type.
　　　* @return retval operation completion status. Returns INVALID_ARGUMENTS
　　　*         if there is no corresponding hardware module found,
　　　*         NOT_INITIALIZED if an error occured while opening the hardware
　　　*         module.
　　　* @return result the interface for the created device.
　　　*/
　　　openDevice(Device device) generates (Result retval, IDevice result);  // 定义一个方法
};
```

## 注册服务
HIDL 接口服务器（实现接口的对象）可注册为已命名的服务。注册的名称不需要与接口或软件包名称相关。

如果没有指定名称，则使用名称“默认”；这应该用于不需要注册同一接口的两个实现的 HAL。

例如，在每个接口中定义的服务注册的 C++ 调用是：
```c++
status_t status = myFoo->registerAsService();
status_t anotherStatus = anotherFoo->registerAsService("another_foo_service");  // if needed
```
HIDL 接口的版本包含在接口本身中。版本自动与服务注册关联，并可通过每个 HIDL 接口上的方法调用 (android::hardware::IInterface::getInterfaceVersion()) 进行检索。服务器对象不需要注册，并可通过 HIDL 方法参数传递到其他进程，相应的接收进程会向服务器发送 HIDL 方法调用。

## 发现服务
客户端代码按名称和版本请求指定的接口，并对所需的 HAL 类调用 getService：
```C++
// C++
sp<V1_1::IFooService> service = V1_1::IFooService::getService();
sp<V1_1::IFooService> alternateService = 1_1::IFooService::getService("another_foo_service");
```
```java
// Java
V1_1.IFooService; service = V1_1.IFooService.getService(true /* retry */);
V1_1.IFooService; alternateService = 1_1.IFooService.getService("another", true /* retry */);
```
每个版本的 HIDL 接口都会被视为单独的接口。因此，IFooService 版本 1.1 和 IFooService 版本 2.2 都可以注册为“foo_service”，并且两个接口上的 getService("foo_service") 都可获取该接口的已注册服务。因此，在大多数情况下，注册或发现服务均无需提供名称参数（也就是说名称为“默认”）。

## vndservicemanager
以前，Binder 服务通过 servicemanager 注册，其他进程可从中检索这些服务。在 Android O 中，servicemanager 现在专用于框架和应用进程，供应商进程无法再对其进行访问。

不过，供应商服务现在可以使用 vndservicemanager，这是一个使用 /dev/vndbinder（作为构建基础的源代码与框架 servicemanager 相同）而非 /dev/binder 的 servicemanager 的新实例。供应商进程无需更改即可与 vndservicemanager 通信；当供应商进程打开 /dev/vndbinder 时，服务查询会自动转至 vndservicemanager。

vndservicemanager 二进制文件包含在 Android 的默认设备 Makefile 中。

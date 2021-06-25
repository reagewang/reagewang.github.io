测试机器版本
android8.1 api27 ABI:arm64-v8a armeabi-v7a armeabi

第一次输出信息
```
I zygote  : Late-enabling -Xcheck:jni
W System  : ClassLoader referenced unknown path:
I QtCore  : Start
I Qt      : qt started
D OpenGLRenderer: HWUI GL Pipeline
I zygote  : Do partial code cache collection, code=26KB, data=23KB
I zygote  : After code cache collection, code=26KB, data=23KB
I zygote  : Increasing code cache capacity to 128KB
I zygote  : Do partial code cache collection, code=62KB, data=56KB
I zygote  : After code cache collection, code=62KB, data=56KB
I zygote  : Increasing code cache capacity to 256KB
I mali_so : [File] : hardware/arm/maliT760/driver/product/base/src/mali_base_kbase.c; [Line] : 947; [Func] : base_context_deal_with_version_affairs_rk_ext;
I mali_so : arm_release_ver of this mali_so is 'r18p0-01rel0', rk_so_ver is '11@0'.
I mali_so : [File] : hardware/arm/maliT760/driver/product/base/src/mali_base_kbase.c; [Line] : 914; [Func] : determine_policy_against_patch_for_sf_memory_leak;
I mali_so : cmdline : org.qtproject.example, we should GO THROUGH patch_for_sf_memory_leak.
D mali_so : [File] : hardware/arm/maliT760/driver/product/base/src/mali_base_kbase.c; [Line] : 955; [Func] : base_context_deal_with_version_affairs_rk_ext;
D mali_so : current process is NOT sf, to bail out.
I zygote  : Do full code cache collection, code=111KB, data=114KB
I zygote  : After code cache collection, code=109KB, data=99KB
I org.qtproject.example: android::hardware::configstore::V1_0::ISurfaceFlingerConfigs::hasWideColorDisplay retrieved: 0
I OpenGLRenderer: Initialized EGL, version 1.4
D OpenGLRenderer: Swap behavior 2
D mali_winsys: EGLint new_window_surface(egl_winsys_display *, void *, EGLSurface, EGLConfig, egl_winsys_surface **, egl_color_buffer_format *, EGLBoolean) returns 0x3000
I [Gralloc]: Got handle 1 for fd 56
I [Gralloc]: leave, w : 1200, h : 1920, format : 0x1,internal_format : 0x1, usage : 0xb00. size=9216000,pixel_stride=1200,byte_stride=4800
I [Gralloc]: leave: prime_fd=56,share_attr_fd=57
I [Gralloc]: Got handle 2 for fd 62
I [Gralloc]: leave, w : 1920, h : 1166, format : 0x1,internal_format : 0x1, usage : 0x933. size=8954880,pixel_stride=1920,byte_stride=7680
I [Gralloc]: leave: prime_fd=62,share_attr_fd=63
I [Gralloc]: Got handle 3 for fd 71
I [Gralloc]: leave, w : 1200, h : 1920, format : 0x1,internal_format : 0x1, usage : 0xb00. size=9216000,pixel_stride=1200,byte_stride=4800
I [Gralloc]: leave: prime_fd=71,share_attr_fd=72
I [Gralloc]: Got handle 4 for fd 65
I [Gralloc]: leave, w : 1920, h : 1166, format : 0x1,internal_format : 0x1, usage : 0x933. size=8954880,pixel_stride=1920,byte_stride=7680
I [Gralloc]: leave: prime_fd=65,share_attr_fd=66
```

环境配置
Qt5.14.1 For Android WIN10配置
qt-opensource-windows-x86-5.14.1.exe安装只选择android模块

安卓三件套
installer_r24.4.1-windows.exe
jdk-8u291-windows-x64.exe
android-ndk-r21e-windows-x86_64.zip

环境变量
D:\Download\android\android-sdk-windows\platform-tools
D:\Download\android\android-ndk-r21e
C:\Program Files\Java\jdk1.8.0_291\bin
C:\Program Files\Java\jre1.8.0_291\bin

编译更改国内源
//阿里仓库
maven { url 'https://maven.aliyun.com/repository/google' }
maven { url 'https://maven.aliyun.com/repository/jcenter'}

```
buildscript {
    repositories {
        //google()
        //jcenter()
        //阿里仓库
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter'}
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
    }
}

repositories {
    //google()
    //jcenter()
    //阿里仓库
    maven { url 'https://maven.aliyun.com/repository/google' }
    maven { url 'https://maven.aliyun.com/repository/jcenter'}
}
```
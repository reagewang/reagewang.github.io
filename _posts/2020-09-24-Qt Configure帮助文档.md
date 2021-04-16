---
layout: post
title: "Qt Configure帮助文档"
tags: [qt]
comments: true
---

```
./configure -help
+ cd qtbase
+ /home/wanghu/Downloads/qt-everywhere-opensource-src-5.9.9/qtbase/configure -top-level -help
Usage:  configure [options] [assignments]

Configure understands variable assignments like VAR=value on the command line. Each uppercased library name (obtainable with -list-libraries) supports the suffixes _INCDIR, _LIBDIR, _PREFIX (INCDIR=PREFIX/include, LIBDIR=PREFIX/lib), _LIBS, and - on Windows and Darwin - _LIBS_DEBUG and _LIBS_RELEASE. E.g., ICU_PREFIX=/opt/icu42 ICU_LIBS="-licui18n -licuuc -licudata".

It is also possible to manipulate any QMAKE_* variable, to amend the values from the mkspec for the build of Qt itself, e.g., QMAKE_CXXFLAGS+=-g3.

Note that the *_LIBS* and QMAKE_* assignments manipulate lists, so items containing meta characters (spaces in particular) need to be quoted according to qmake rules. On top of that, the assignments as a whole need to be quoted according to shell rules. It is recommended to use single quotes for the inner quoting and double quotes for the outer quoting.

Top-level installation directories:
  -prefix <dir> ...... The deployment directory, as seen on the target device. [/usr/local/Qt-$QT_VERSION, $PWD if -developer-build]
  -extprefix <dir> ... The installation directory, as seen on the host machine. [SYSROOT/PREFIX]
  -hostprefix [dir] .. The installation directory for build tools running on the host machine. If [dir] is not given, the current build directory will be used. [EXTPREFIX]
  -external-hostbindir <path> ... Path to Qt tools built for this machine. Use this when -platform does not match the current system, i.e., to make a Canadian Cross Build.

Fine tuning of installation directory layout. Note that all directories except -sysconfdir should be located under -prefix/-hostprefix:

  -bindir <dir> ......... Executables [PREFIX/bin]
  -headerdir <dir> ...... Header files [PREFIX/include]
  -libdir <dir> ......... Libraries [PREFIX/lib]
  -archdatadir <dir> .... Arch-dependent data [PREFIX]
  -plugindir <dir> ...... Plugins [ARCHDATADIR/plugins]
  -libexecdir <dir> ..... Helper programs [ARCHDATADIR/bin on Windows, ARCHDATADIR/libexec otherwise]
  -importdir <dir> ...... QML1 imports [ARCHDATADIR/imports]
  -qmldir <dir> ......... QML2 imports [ARCHDATADIR/qml]
  -datadir <dir> ........ Arch-independent data [PREFIX]
  -docdir <dir> ......... Documentation [DATADIR/doc]
  -translationdir <dir> . Translations [DATADIR/translations]
  -sysconfdir <dir> ..... Settings used by Qt programs [PREFIX/etc/xdg]
  -examplesdir <dir> .... Examples [PREFIX/examples]
  -testsdir <dir> ....... Tests [PREFIX/tests]

  -hostbindir <dir> ..... Host executables [HOSTPREFIX/bin]
  -hostlibdir <dir> ..... Host libraries [HOSTPREFIX/lib]
  -hostdatadir <dir> .... Data used by qmake [HOSTPREFIX]

Conventions for the remaining options: When an option's description is followed by a list of values in brackets, the interpretation is as follows: 'yes' represents the bare option; all other values are possible prefixes to the option, e.g., -no-gui. Alternatively, the value can be assigned, e.g., --gui=yes. Values are listed in the order they are tried if not specified; 'auto' is a shorthand for 'yes/no'. Solitary 'yes' and 'no' represent binary options without auto-detection.

Configure meta:

  -help, -h ............ Display this help screen
  -verbose, -v ......... Print verbose messages during configuration 在配置过程中打印详细消息
  -continue ............ Continue configure despite errors 发生错误，继续编译
  -redo ................ Re-configure with previously used options. Additional options may be passed, but will not be saved for later use by -redo.
  -recheck [test,...] .. Discard cached negative configure test results. Use this after installing missing dependencies. Alternatively, if tests are specified, only their results are discarded.
  -recheck-all ......... Discard all cached configure test results.

  -feature-<feature> ... Enable <feature>
  -no-feature-<feature> ... Disable <feature> [none]
  -list-features ....... List available features. Note that some features have dedicated command line options as well. 可用功能列表。注意，有些特性也有专门的命令行选项。

  -list-libraries ...... List possible external dependencies. 列出可能的外部依赖关系。

Build options:

  -opensource .......... Build the Open-Source Edition of Qt 构建开源版本
  -commercial .......... Build the Commercial Edition of Qt 构建商业版
  -confirm-license ..... Automatically acknowledge the license 自动确认许可证

  -release ............. Build Qt with debugging turned off [yes] 构建发布版
  -debug ............... Build Qt with debugging turned on [no] 构建调试版
  -debug-and-release ... Build two versions of Qt, with and without
                         debugging turned on [yes] (Apple and Windows only)
  -optimize-debug ...... Enable debug-friendly optimizations in debug builds
                         [auto] (Not supported with MSVC or Clang toolchains)
  -optimize-size ....... Optimize release builds for size instead of speed [no]
  -optimized-tools ..... Build optimized host tools even in debug build [no]
  -force-debug-info .... Create symbol files for release builds [no]
  -separate-debug-info . Split off debug information to separate files [no]
  -strip ............... Strip release binaries of unneeded symbols [yes]
  -force-asserts ....... Enable Q_ASSERT even in release builds [no]
  -developer-build ..... Compile and link Qt for developing Qt itself
                         (exports for auto-tests, extra checks, etc.) [no]

  -shared .............. Build shared Qt libraries [yes] (no for UIKit) 构建动态Qt库
  -static .............. Build static Qt libraries [no] (yes for UIKit) 构建静态Qt库
  -framework ........... Build Qt framework bundles [yes] (Apple only) 构建Qt框架包

  -platform <target> ... Select host mkspec [detected]
  -xplatform <target> .. Select target mkspec when cross-compiling [PLATFORM] 选择编译器
  -device <name> ....... Cross-compile for device <name>
  -device-option <key=value> ... Add option for the device mkspec

  -appstore-compliant .. Disable code that is not allowed in platform app stores. This is on by default for platforms which require distribution through an app store by default, in particular Android, iOS, tvOS, watchOS, and Universal Windows Platform. [auto] 禁用平台应用商店中不允许使用的代码。对于那些需要通过默认应用商店进行发行的平台，特别是Android、iOS、tvOS、watchOS和通用Windows平台，这是默认开启的。

  -qtnamespace <name> .. Wrap all Qt library code in 'namespace <name> {...}'.
  -qtlibinfix <infix> .. Rename all libQt5*.so to libQt5*<infix>.so.

  -testcocoon .......... Instrument with the TestCocoon code coverage tool [no]
  -gcov ................ Instrument with the GCov code coverage tool [no]
  -sanitize {address|thread|memory|undefined} ... Instrument with the specified compiler sanitizer. Note that some sanitizers cannot be combined; for example, -sanitize address cannot be combined with -sanitize thread.

  -c++std <edition> .... Select C++ standard <edition> [c++1z/c++14/c++11] (Not supported with MSVC)

  -sse2 ................ Use SSE2 instructions [auto] 使用SSE2指令
  -sse3/-ssse3/-sse4.1/-sse4.2/-avx/-avx2/-avx512 ... Enable use of particular x86 instructions [auto] Enabled ones are still subject to runtime detection. 启用特定的x86指令,启用的指令仍然受运行时检测。
  -mips_dsp/-mips_dspr2 ... Use MIPS DSP/rev2 instructions [auto] 使用MIPS DSP/rev2指令

  -qreal <type> ........ typedef qreal to the specified type. [double] Note: this affects binary compatibility.

  -R <string> .......... Add an explicit runtime library path to the Qt libraries. Supports paths relative to LIBDIR. 向Qt库添加一个显式的运行时库路径。支持相对于LIBDIR的路径。
  -rpath ............... Link Qt libraries and executables using the library install path as a runtime library path. Similar to -R LIBDIR. On Apple platforms, disabling this implies using absolute install names (based in LIBDIR) for dynamic libraries and frameworks. [auto] 使用库安装路径作为运行时库路径链接Qt库和可执行文件。类似于-R LIBDIR。在Apple平台上，禁用此功能意味着对动态库和框架使用绝对安装名(基于LIBDIR)。

  -reduce-exports ...... Reduce amount of exported symbols [auto] 减少输出符号的数量
  -reduce-relocations .. Reduce amount of relocations [auto] (Unix only)

  -plugin-manifests .... Embed manifests into plugins [no] (Windows only)
  -static-runtime ...... With -static, use static runtime [no] (Windows only)

  -pch ................. Use precompiled headers [auto] 使用预编译头
  -ltcg ................ Use Link Time Code Generation [no]
  -use-gold-linker ..... Use the GNU gold linker [auto] 使用GNU链接器
  -incredibuild-xge .... Use the IncrediBuild XGE [no] (Windows only)
  -ccache .............. Use the ccache compiler cache [no] (Unix only)
  -make-tool <tool> .... Use <tool> to build qmake [nmake] (Windows only)
  -mp .................. Use multiple processors for compilation (MSVC only)

  -warnings-are-errors . Treat warnings as errors [no; yes if -developer-build]
  -silent .............. Reduce the build output so that warnings and errors can be seen more easily 减少构建输出，以便更容易看到警告和错误

Build environment:

  -sysroot <dir> ....... Set <dir> as the target sysroot
  -gcc-sysroot ......... With -sysroot, pass --sysroot to the compiler [yes]

  -pkg-config .......... Use pkg-config [auto] (Unix only)

  -D <string> .......... Pass additional preprocessor define
  -I <string> .......... Pass additional include path
  -L <string> .......... Pass additional library path
  -F <string> .......... Pass additional framework path (Apple only)

  -sdk <sdk> ........... Build Qt using Apple provided SDK <sdk>. The argument should be one of the available SDKs as listed by 'xcodebuild -showsdks'. Note that the argument applies only to Qt libraries and applications built using the target mkspec - not host tools such as qmake, moc, rcc, etc.

  -android-sdk path .... Set Android SDK root path [$ANDROID_SDK_ROOT]
  -android-ndk path .... Set Android NDK root path [$ANDROID_NDK_ROOT]
  -android-ndk-platform  Set Android platform
  -android-ndk-host .... Set Android NDK host (linux-x86, linux-x86_64, etc.) [$ANDROID_NDK_HOST]
  -android-arch ........ Set Android architecture (armeabi, armeabi-v7a, arm64-v8a, x86, x86_64, mips, mips64)
  -android-toolchain-version ... Set Android toolchain version
  -android-style-assets ... Automatically extract style assets from the device at run time. This option makes the Android style behave correctly, but also makes the Android platform plugin incompatible with the LGPL2.1. [yes]

Component selection:

  -skip <repo> ......... Exclude an entire repository from the build.
  -make <part> ......... Add <part> to the list of parts to be built. Specifying this option clears the default list first. [libs and examples, also tools if not cross-building, also tests if -developer-build]
  -nomake <part> ....... Exclude <part> from the list of parts to be built.
  -compile-examples .... When unset, install only the sources of examples [yes]
  -gui ................. Build the Qt GUI module and dependencies [yes]
  -widgets ............. Build the Qt Widgets module and dependencies [yes]
  -no-dbus ............. Do not build the Qt D-Bus module [default on Android and Windows]
  -dbus-linked ......... Build Qt D-Bus and link to libdbus-1 [auto]
  -dbus-runtime ........ Build Qt D-Bus and dynamically load libdbus-1 [no]
  -accessibility ....... Enable accessibility support [yes] Note: Disabling accessibility is not recommended.
  -qml-debug ........... Enable QML debugging support [yes]

Qt comes with bundled copies of some 3rd party libraries. These are used by default if auto-detection of the respective system library fails.

Core options:

  -doubleconversion .... Select used double conversion library [system/qt/no] No implies use of sscanf_l and snprintf_l (imprecise).
  -glib ................ Enable Glib support [no; auto on Unix]
  -eventfd ............. Enable eventfd support
  -inotify ............. Enable inotify support
  -iconv ............... Enable iconv(3) support [posix/sun/gnu/no] (Unix only)
  -icu ................. Enable ICU support [auto]
  -pcre ................ Select used libpcre2 [system/qt]
  -pps ................. Enable PPS support [auto] (QNX only)
  -zlib ................ Select used zlib [system/qt]

  Logging backends:
    -journald .......... Enable journald support [no] (Unix only)
    -syslog ............ Enable syslog support [no] (Unix only)
    -slog2 ............. Enable slog2 support [auto] (QNX only)

Network options:

  -ssl ................. Enable either SSL support method [auto]
  -no-openssl .......... Do not use OpenSSL [default on Apple and WinRT]
  -openssl-linked ...... Use OpenSSL and link to libssl [no]
  -openssl-runtime ..... Use OpenSSL and dynamically load libssl [auto]
  -securetransport ..... Use SecureTransport [auto] (Apple only)

  -sctp ................ Enable SCTP support [no]

  -libproxy ............ Enable use of libproxy [no]
  -system-proxies ...... Use system network proxies by default [yes]

Gui, printing, widget options:

  -cups ................ Enable CUPS support [auto] (Unix only)

  -fontconfig .......... Enable Fontconfig support [auto] (Unix only)
  -freetype ............ Select used FreeType [system/qt/no]
  -harfbuzz ............ Select used HarfBuzz-NG [system/qt/no] (Not auto-detected on Apple and Windows)

  -gtk ................. Enable GTK platform theme support [auto]

  -lgmon ............... Enable lgmon support [auto] (QNX only)

  -no-opengl ........... Disable OpenGL support
  -opengl <api> ........ Enable OpenGL support. Supported APIs: es2 (default on Windows), desktop (default on Unix), dynamic (Windows only)
  -opengles3 ........... Enable OpenGL ES 3.x support instead of ES 2.x [auto]
  -angle ............... Use bundled ANGLE to support OpenGL ES 2.0 [auto] (Windows only)
  -combined-angle-lib .. Merge LibEGL and LibGLESv2 into LibANGLE (Windows only)

  -qpa <name> .......... Select default QPA backend (e.g., xcb, cocoa, windows)
  -xcb-xlib............. Enable Xcb-Xlib support [auto]

  Platform backends:
    -direct2d .......... Enable Direct2D support [auto] (Windows only)
    -directfb .......... Enable DirectFB support [no] (Unix only)
    -eglfs ............. Enable EGLFS support [auto; no on Android and Windows]
    -gbm ............... Enable backends for GBM [auto] (Linux only)
    -kms ............... Enable backends for KMS [auto] (Linux only)
    -linuxfb ........... Enable Linux Framebuffer support [auto] (Linux only)
    -mirclient ......... Enable Mir client support [no] (Linux only)
    -xcb ............... Select used xcb-* libraries [system/qt/no] (-qt-xcb still uses system version of libxcb itself)

  Input backends:
    -evdev ............. Enable evdev support [auto]
    -imf ............... Enable IMF support [auto] (QNX only)
    -libinput .......... Enable libinput support [auto]
    -mtdev ............. Enable mtdev support [auto]
    -tslib ............. Enable tslib support [auto]
    -xinput2 ........... Enable XInput2 support [auto]
    -xkbcommon-x11 ..... Select xkbcommon used in combination with xcb [system/qt/no]
    -xkb-config-root <dir> ... With -qt-xkbcommon-x11, set default XKB config root <dir> [detect]
    -xkbcommon-evdev ... Enable X-less xkbcommon in combination with libinput [auto]

  Image formats:
    -gif ............... Enable reading support for GIF [auto]
    -ico ............... Enable support for ICO [yes]
    -libpng ............ Select used libpng [system/qt/no]
    -libjpeg ........... Select used libjpeg [system/qt/no]

Database options:

  -sql-<driver> ........ Enable SQL <driver> plugin. Supported drivers: db2 ibase mysql oci odbc psql sqlite2 sqlite tds [all auto]
  -sqlite .............. Select used sqlite3 [system/qt]

Qt3D options:

  -assimp .............. Select used assimp library [system/qt/no]
  -qt3d-profile-jobs ... Enable jobs profiling [no]
  -qt3d-profile-gl ..... Enable OpenGL profiling [no]

Multimedia options:

  -pulseaudio .......... Enable PulseAudio support [auto] (Unix only)
  -alsa ................ Enable ALSA support [auto] (Unix only)
  -no-gstreamer ........ Disable support for GStreamer
  -gstreamer [version] . Enable GStreamer support [auto] With no parameter, 1.0 is tried first, then 0.10.
  -mediaplayer-backend <name> ... Select media player backend (Windows only) Supported backends: directshow (default), wmf

Webengine options:

  -alsa ................ Enable ALSA support [auto] (Linux only)
  -webengine-icu ....... Select used ICU libraries [system/qt] (Linux only)
  -ffmpeg .............. Select used FFmpeg libraries [system/qt] (Linux only)
  -opus ................ Select used Opus libraries [system/qt] (Linux only)
  -webp ................ Select used WebP libraries [system/qt] (Linux only)
  -pepper-plugins ...... Enable use of Pepper Flash and Widevine plugins [auto]
  -printing-and-pdf .... Enable use of printing and output to PDF [auto]
  -proprietary-codecs .. Enable support for proprietary codecs [no]
  -pulseaudio .......... Enable PulseAudio support [auto] (Linux only)
  -spellchecker ........ Enable support for spellchecker [yes]
  -webrtc .............. Enable support for WebRTC [auto]
```
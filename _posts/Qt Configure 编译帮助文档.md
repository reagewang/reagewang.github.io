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
顶级安装目录
  -prefix <dir> ...... The deployment directory, as seen on the target device. [/usr/local/Qt-$QT_VERSION, $PWD if -developer-build] 指定部署目录
  -extprefix <dir> ... The installation directory, as seen on the host machine. [SYSROOT/PREFIX] 安装目录
  -hostprefix [dir] .. The installation directory for build tools running on the host machine. If [dir] is not given, the current build directory will be used. [EXTPREFIX]
  -external-hostbindir <path> ... Path to Qt tools built for this machine. Use this when -platform does not match the current system, i.e., to make a Canadian Cross Build.

Fine tuning of installation directory layout. Note that all directories except -sysconfdir should be located under -prefix/-hostprefix:
安装目录布局的微调。注意，除-sysconfdir外的所有目录都应该位于-prefix/-hostprefix下

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

  -help, -h ............ Display this help screen. 显示帮助文档
  -verbose, -v ......... Print verbose messages during configuration 在配置过程中打印详细消息
  -continue ............ Continue configure despite errors 发生错误，继续编译
  -redo ................ Re-configure with previously used options. Additional options may be passed, but will not be saved for later use by -redo.
  -recheck [test,...] .. Discard cached negative configure test results. Use this after installing missing dependencies. Alternatively, if tests are specified, only their results are discarded.
  -recheck-all ......... Discard all cached configure test results. 释放缓存重新编译

  -feature-<feature> ... Enable <feature> 启用特性
  -no-feature-<feature> ... Disable <feature> [none] 禁用特性
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
  -strip ............... Strip release binaries of unneeded symbols [yes] 删除多余的符号
  -force-asserts ....... Enable Q_ASSERT even in release builds [no]
  -developer-build ..... Compile and link Qt for developing Qt itself
                         (exports for auto-tests, extra checks, etc.) [no]

  -shared .............. Build shared Qt libraries [yes] (no for UIKit) 构建动态Qt库
  -static .............. Build static Qt libraries [no] (yes for UIKit) 构建静态Qt库
  -framework ........... Build Qt framework bundles [yes] (Apple only) 构建Qt框架包

  -platform <target> ... Select host mkspec [detected] 选择主机平台
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

  -qreal <type> ........ typedef qreal to the specified type. [double] Note: this affects binary compatibility. 将qreal定义为指定的类型。注意:这会影响二进制兼容性。

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
  -ccache .............. Use the ccache compiler cache [no] (Unix only) 使用编译器缓存
  -make-tool <tool> .... Use <tool> to build qmake [nmake] (Windows only)
  -mp .................. Use multiple processors for compilation (MSVC only) 使用多个处理器进行编译

  -warnings-are-errors . Treat warnings as errors [no; yes if -developer-build] 将警告视为错误
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

```
./configure -list-features
+ cd qtbase
+ /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/configure -top-level -list-features
Creating qmake...
.Done.

abstractbutton .......... Widgets: Abstract base class of button widgets, providing functionality common to buttons.
abstractslider .......... Widgets: Common super class for widgets like QScrollBar, QSlider and QDial.
accessibility ........... Utilities: Provides accessibility support.
action .................. Kernel: Provides widget actions.
animation ............... Utilities: Provides a framework for animations.
appstore-compliant ...... Disables code that is not allowed in platform app stores
bearermanagement ........ Networking: Provides bearer management for the network stack.
big_codecs .............. Internationalization: Supports big codecs, e.g. CJK.
buttongroup ............. Widgets: Supports organizing groups of button widgets.
calendarwidget .......... Widgets: Provides a monthly based calendar widget allowing the user to select a date.
checkbox ................ Widgets: Provides a checkbox with a text label.
clipboard ............... Kernel: Provides cut and paste operations.
codecs .................. Internationalization: Supports non-unicode text conversions.
colordialog ............. Dialogs: Provides a dialog widget for specifying colors.
colornames .............. Painting: Supports color names such as "red", used by QColor and by some HTML documents.
columnview .............. ItemViews: Provides a model/view implementation of a column view.
combobox ................ Widgets: Provides drop-down boxes presenting a list of options to the user.
commandlineparser ....... Utilities: Provides support for command line parsing.
commandlinkbutton ....... Widgets: Provides a Vista style command link button.
completer ............... Utilities: Provides completions based on an item model.
concurrent .............. Kernel: Provides a high-level multi-threading API.
contextmenu ............. Widgets: Adds pop-up menus on right mouse click to numerous widgets.
cssparser ............... Kernel: Provides a parser for Cascading Style Sheets.
cups .................... Painting: Provides support for the Common Unix Printing System.
cursor .................. Kernel: Provides mouse cursors.
d3d12 ................... Qt Quick: Provides a Direct3D 12 backend for the scenegraph.
datawidgetmapper ........ ItemViews: Provides mapping between a section of a data model to widgets.
datestring .............. Data structures: Provides conversion between dates and strings.
datetimeedit ............ Widgets: Supports editing dates and times.
datetimeparser .......... Utilities: Provides support for parsing date-time texts.
desktopservices ......... Utilities: Provides methods for accessing common desktop services.
dial .................... Widgets: Provides a rounded range control, e.g., like a speedometer.
dialog .................. Dialogs: Base class of dialog windows.
dialogbuttonbox ......... Dialogs: Presents buttons in a layout that is appropriate for the current widget style.
dirmodel ................ ItemViews: Provides a data model for the local filesystem.
dnslookup ............... Networking: Provides API for DNS lookups.
dockwidget .............. Widgets: Supports docking widgets inside a QMainWindow or floated as a top-level window on the desktop.
dom ..................... File I/O: Supports the Document Object Model.
draganddrop ............. Kernel: Supports the drag and drop mechansim.
dtls .................... Networking: Provides a DTLS implementation
effects ................. Kernel: Provides special widget effects (e.g. fading and scrolling).
errormessage ............ Dialogs: Provides an error message display dialog.
filedialog .............. Dialogs: Provides a dialog widget for selecting files or directories.
filesystemiterator ...... File I/O: Provides fast file system iteration.
filesystemmodel ......... File I/O: Provides a data model for the local filesystem.
filesystemwatcher ....... File I/O: Provides an interface for monitoring files and directories for modifications.
fontcombobox ............ Widgets: Provides a combobox that lets the user select a font family.
fontdialog .............. Dialogs: Provides a dialog widget for selecting fonts.
formlayout .............. Widgets: Manages forms of input widgets and their associated labels.
freetype ................ Fonts: Supports the FreeType 2 font engine (and its supported font formats).
fscompleter ............. Utilities: Provides file name completion in QFileDialog.
ftp ..................... Networking: Provides support for the File Transfer Protocol in QNetworkAccessManager.
future .................. Kernel: Provides QFuture and related classes.
geoservices_esri ........ Location: Provides access to Esri geoservices
geoservices_here ........ Location: Provides access to HERE geoservices
geoservices_itemsoverlay . Location: Provides access to the itemsoverlay maps
geoservices_mapbox ...... Location: Provides access to Mapbox geoservices
geoservices_mapboxgl .... Location: Provides access to the Mapbox vector maps
geoservices_osm ......... Location: Provides access to OpenStreetMap geoservices
gestures ................ Utilities: Provides a framework for gestures.
graphicseffect .......... Widgets: Provides various graphics effects.
graphicsview ............ Widgets: Provides a canvas/sprite framework.
groupbox ................ Widgets: Provides widget grouping boxes with frames.
highdpiscaling .......... Kernel: Provides automatic scaling of DPI-unaware applications on high-DPI displays.
http .................... Networking: Provides support for the Hypertext Transfer Protocol in QNetworkAccessManager.
iconv ................... Internationalization: Provides internationalization on Unix.
identityproxymodel ...... ItemViews: Supports proxying a source model unmodified.
im ...................... Kernel: Provides complex input methods.
image_heuristic_mask .... Images: Supports creating a 1-bpp heuristic mask for images.
image_text .............. Images: Supports image file text strings.
imageformat_bmp ......... Images: Supports Microsoft's Bitmap image file format.
imageformat_jpeg ........ Images: Supports the Joint Photographic Experts Group image file format.
imageformat_png ......... Images: Supports the Portable Network Graphics image file format.
imageformat_ppm ......... Images: Supports the Portable Pixmap image file format.
imageformat_xbm ......... Images: Supports the X11 Bitmap image file format.
imageformat_xpm ......... Images: Supports the X11 Pixmap image file format.
imageformatplugin ....... Images: Provides a base for writing a image format plugins.
inputdialog ............. Dialogs: Provides a simple convenience dialog to get a single value from the user.
itemmodel ............... ItemViews: Provides the item model for item views
itemmodeltester ......... Provides a utility to test item models.
itemviews ............... ItemViews: Provides the model/view architecture managing the relationship between data and the way it is presented to the user.
keysequenceedit ......... Widgets: Provides a widget for editing QKeySequences.
label ................... Widgets: Provides a text or image display.
lcdnumber ............... Widgets: Provides LCD-like digits.
library ................. File I/O: Provides a wrapper for dynamically loaded libraries.
lineedit ................ Widgets: Provides single-line edits.
listview ................ ItemViews: Provides a list or icon view onto a model.
listwidget .............. Widgets: Provides item-based list widgets.
localserver ............. Networking: Provides a local socket based server.
location-labs-plugin .... Location: Provides experimental QtLocation QML types
mainwindow .............. Widgets: Provides main application windows.
mdiarea ................. Widgets: Provides an area in which MDI windows are displayed.
menu .................... Widgets: Provides popup-menus.
menubar ................. Widgets: Provides pull-down menu items.
messagebox .............. Dialogs: Provides message boxes displaying informative messages and simple questions.
mimetype ................ Utilities: Provides MIME type handling.
movie ................... Images: Supports animated images.
multiprocess ............ Utilities: Provides support for detecting the desktop environment, launching external processes and opening URLs.
networkdiskcache ........ Networking: Provides a disk cache for network resources.
networkinterface ........ Networking: Supports enumerating a host's IP addresses and network interfaces.
networkproxy ............ Networking: Provides network proxy support.
paint_debug ............. Painting: Enabled debugging painting with the environment variables QT_FLUSH_UPDATE and QT_FLUSH_PAINT.
pdf ..................... Painting: Provides a PDF backend for QPainter.
picture ................. Painting: Supports recording and replaying QPainter commands.
printdialog ............. Dialogs: Provides a dialog widget for specifying printer configuration.
printer ................. Painting: Provides a printer backend of QPainter.
printpreviewdialog ...... Dialogs: Provides a dialog for previewing and configuring page layouts for printer output.
printpreviewwidget ...... Widgets: Provides a widget for previewing page layouts for printer output.
process ................. File I/O: Supports external process invocation.
processenvironment ...... File I/O: Provides a higher-level abstraction of environment variables.
progressbar ............. Widgets: Supports presentation of operation progress.
progressdialog .......... Dialogs: Provides feedback on the progress of a slow operation.
properties .............. Kernel: Supports scripting Qt-based applications.
proxymodel .............. ItemViews: Supports processing of data passed between another model and a view.
pushbutton .............. Widgets: Provides a command button.
qml-animation ........... QML: Provides support for animations and timers in QML.
qml-debug ............... QML: Provides infrastructure and plugins for debugging and profiling.
qml-delegate-model ...... QML: Provides the DelegateModel QML type.
qml-devtools ............ QML: Provides the QmlDevtools library and various utilities.
qml-list-model .......... QML: Provides the ListModel QML type.
qml-locale .............. QML: Provides support for locales in QML.
qml-network ............. QML: Provides network transparency.
qml-preview ............. QML: Updates QML documents in your application live as you change them on disk
qml-profiler ............ QML: Supports retrieving QML tracing data from an application.
qml-sequence-object ..... QML: Supports mapping sequence types into QML.
qml-worker-script ....... QML: Enables the use of threads in QML.
qml-xml-http-request .... QML: Provides support for sending XML http requests.
qt3d-animation .......... Aspects: Use the 3D Animation Aspect library
qt3d-extras ............. Aspects: Use the 3D Extra library
qt3d-input .............. Aspects: Use the 3D Input Aspect library
qt3d-logic .............. Aspects: Use the 3D Logic Aspect library
qt3d-opengl-renderer .... Qt 3D Renderers: Use the OpenGL renderer
qt3d-render ............. Aspects: Use the 3D Render Aspect library
qt3d-simd-avx2 .......... Use AVX2 SIMD instructions to accelerate matrix operations
qt3d-simd-sse2 .......... Use SSE2 SIMD instructions to accelerate matrix operations
quick-animatedimage ..... Qt Quick: Provides the AnimatedImage item.
quick-canvas ............ Qt Quick: Provides the Canvas item.
quick-designer .......... Qt Quick: Provides support for the Qt Quick Designer in Qt Creator.
quick-flipable .......... Qt Quick: Provides the Flipable item.
quick-gridview .......... Qt Quick: Provides the GridView item.
quick-listview .......... Qt Quick: Provides the ListView item.
quick-particles ......... Qt Quick: Provides a particle system.
quick-path .............. Qt Quick: Provides Path elements.
quick-pathview .......... Qt Quick: Provides the PathView item.
quick-positioners ....... Qt Quick: Provides Positioner items.
quick-repeater .......... Qt Quick: Provides the Repeater item.
quick-shadereffect ...... Qt Quick: Provides Shader effects.
quick-sprite ............ Qt Quick: Provides the Sprite item.
quick-tableview ......... Qt Quick: Provides the TableView item.
quickcontrols2-fusion ... Quick Controls 2: Provides the platform agnostic desktop-oriented Fusion style.
quickcontrols2-imagine .. Quick Controls 2: Provides a style based on configurable image assets.
quickcontrols2-material . Quick Controls 2: Provides a style based on the Material Design guidelines.
quickcontrols2-universal . Quick Controls 2: Provides a style based on the Universal Design guidelines.
quicktemplates2-hover ... Quick Templates 2: Provides support for hover effects.
quicktemplates2-multitouch . Quick Templates 2: Provides support for multi-touch.
radiobutton ............. Widgets: Provides a radio button with a text label.
regularexpression ....... Kernel: Provides an API to Perl-compatible regular expressions.
resizehandler ........... Widgets: Provides an internal resize handler for dock widgets.
rubberband .............. Widgets: Supports using rubberbands to indicate selections and boundaries.
scrollarea .............. Widgets: Supports scrolling views onto widgets.
scrollbar ............... Widgets: Provides scrollbars allowing the user access parts of a document that is larger than the widget used to display it.
scroller ................ Widgets: Enables kinetic scrolling for any scrolling widget or graphics item.
scxml-ecmascriptdatamodel . SCXML: Enables the usage of ecmascript data models in SCXML state machines.
sessionmanager .......... Kernel: Provides an interface to the windowing system's session management.
settings ................ File I/O: Provides persistent application settings.
sha3-fast ............... Utilities: Optimizes SHA3 for speed instead of size.
sharedmemory ............ Kernel: Provides access to a shared memory segment.
shortcut ................ Kernel: Provides keyboard accelerators and shortcuts.
sizegrip ................ Widgets: Provides corner-grips for resizing top-level windows.
slider .................. Widgets: Provides sliders controlling a bounded value.
socks5 .................. Networking: Provides SOCKS5 support in QNetworkProxy.
sortfilterproxymodel .... ItemViews: Supports sorting and filtering of data passed between another model and a view.
spinbox ................. Widgets: Provides spin boxes handling integers and discrete sets of values.
splashscreen ............ Widgets: Supports splash screens that can be shown during application startup.
splitter ................ Widgets: Provides user controlled splitter widgets.
sqlmodel ................ Provides item model classes backed by SQL databases.
stackedwidget ........... Widgets: Provides stacked widgets.
standarditemmodel ....... ItemViews: Provides a generic model for storing custom data.
statemachine ............ Utilities: Provides hierarchical finite state machines.
statusbar ............... Widgets: Supports presentation of status information.
statustip ............... Widgets: Supports status tip functionality and events.
stringlistmodel ......... ItemViews: Provides a model that supplies strings to views.
style-stylesheet ........ Styles: Provides a widget style which is configurable via CSS.
syntaxhighlighter ....... Widgets: Supports custom syntax highlighting.
systemsemaphore ......... Kernel: Provides a general counting system semaphore.
systemtrayicon .......... Utilities: Provides an icon for an application in the system tray.
tabbar .................. Widgets: Provides tab bars, e.g., for use in tabbed dialogs.
tabletevent ............. Kernel: Supports tablet events.
tableview ............... ItemViews: Provides a default model/view implementation of a table view.
tablewidget ............. Widgets: Provides item-based table views.
tabwidget ............... Widgets: Supports stacking tabbed widgets.
temporaryfile ........... File I/O: Provides an I/O device that operates on temporary files.
textbrowser ............. Widgets: Supports HTML document browsing.
textcodec ............... Internationalization: Supports conversions between text encodings.
textdate ................ Data structures: Supports month and day names in dates.
textedit ................ Widgets: Supports rich text editing.
texthtmlparser .......... Kernel: Provides a parser for HTML.
textodfwriter ........... Kernel: Provides an ODF writer.
thread .................. Kernel: Provides QThread and related classes.
timezone ................ Utilities: Provides support for time-zone handling.
toolbar ................. Widgets: Provides movable panels containing a set of controls.
toolbox ................. Widgets: Provides columns of tabbed widget items.
toolbutton .............. Widgets: Provides quick-access buttons to commands and options.
tooltip ................. Widgets: Supports presentation of tooltips.
topleveldomain .......... Utilities: Provides support for extracting the top level domain from URLs.
translation ............. Internationalization: Supports translations using QObject::tr().
treeview ................ ItemViews: Provides a default model/view implementation of a tree view.
treewidget .............. Widgets: Provides views using tree models.
tuiotouch ............... Provides the TuioTouch input plugin.
udpsocket ............... Networking: Provides access to UDP sockets.
undocommand ............. Utilities: Applies (redo or) undo of a single change in a document.
undogroup ............... Utilities: Provides the ability to cluster QUndoCommands.
undostack ............... Utilities: Provides the ability to (redo or) undo a list of changes in a document.
undoview ................ Utilities: Provides a widget which shows the contents of an undo stack.
validator ............... Widgets: Supports validation of input text.
wayland-compositor-quick . Allows QtWayland compositor types to be used with QtQuick
webengine-embedded-build . WebEngine: Enables the embedded build configuration.
webengine-full-debug-info . Enables debug information for Blink and V8.
webengine-kerberos ...... WebEngine: Enables Kerberos Authentication Support
webengine-native-spellchecker . WebEngine: Use the system's native spellchecking engine.
webengine-pepper-plugins . WebEngine: Enables use of Pepper Flash plugins.
webengine-printing-and-pdf . WebEngine: Provides printing and output to PDF.
webengine-proprietary-codecs . WebEngine: Enables the use of proprietary codecs such as h.264/h.265 and MP3.
webengine-spellchecker .. WebEngine: Provides a spellchecker.
webengine-v8-snapshot ... Enables the v8 snapshot, for fast v8 context creation
webengine-webchannel .... WebEngine: Provides QtWebChannel integration.
webengine-webrtc ........ WebEngine: Provides WebRTC support.
whatsthis ............... Widget Support: Supports displaying "What's this" help.
wheelevent .............. Kernel: Supports wheel events.
widgettextcontrol ....... Widgets: Provides text control functionality to other widgets.
wizard .................. Dialogs: Provides a framework for multi-page click-through dialogs.
xml-schema .............. QtXmlPatterns: Provides XML schema validation.
xmlstream ............... Kernel: Provides a simple streaming API for XML.
xmlstreamreader ......... Kernel: Provides a well-formed XML parser with a simple streaming API.
xmlstreamwriter ......... Kernel: Provides a XML writer with a simple streaming API.
```

```
./configure -所有指令
-accessibility                                       -- ....... Enable accessibility support (yes)
-alsa                                                -- ................ Enable ALSA support (auto) (Unix only)
-android-arch                                        -- ........ Set Android architecture (armeabi, armeabi-v7a,
-android-ndk                                         -- path .... Set Android NDK root path ($ANDROID_NDK_ROOT)
-android-ndk-host                                    -- .... Set Android NDK host (linux-x86, linux-x86_64, etc.)
-android-ndk-platform                                -- Set Android platform
-android-sdk                                         -- path .... Set Android SDK root path ($ANDROID_SDK_ROOT)
-android-style-assets                                -- Automatically extract style assets from the device at
-android-toolchain-version                           -- ... Set Android toolchain version
-angle                                               -- ............... Use bundled ANGLE to support OpenGL ES 2.0 (auto)
-appstore-compliant                                  -- .. Disable code that is not allowed in platform app stores.
-archdatadir                                         -- <dir> .... Arch-dependent data (PREFIX)
-assimp                                              -- .............. Select used assimp library (system/qt/no)
-bindir                                              -- <dir> ......... Executables (PREFIX/bin)
-ccache                                              -- .............. Use the ccache compiler cache (no) (Unix only)
-combined-angle-lib                                  -- .. Merge LibEGL and LibGLESv2 into LibANGLE (Windows only)
-commercial                                          -- .......... Build the Commercial Edition of Qt
-compile-examples                                    -- .... When unset, install only the sources of examples
-confirm-license                                     -- ..... Automatically acknowledge the license
-continue                                            -- ............ Continue configure despite errors
-c++std                                              -- <edition> .... Select C++ standard <edition> (c++1z/c++14/c++11)
-cups                                                -- ................ Enable CUPS support (auto) (Unix only)
-D                                                   -- <string> .......... Pass additional preprocessor define
-datadir                                             -- <dir> ........ Arch-independent data (PREFIX)
-dbus-linked                                         -- ......... Build Qt D-Bus and link to libdbus-1 (auto)
-dbus-runtime                                        -- ........ Build Qt D-Bus and dynamically load libdbus-1 (no)
-debug                                               -- ............... Build Qt with debugging turned on (no)
-debug-and-release                                   -- ... Build two versions of Qt, with and without
-developer-build                                     -- ..... Compile and link Qt for developing Qt itself
-device                                              -- <name> ....... Cross-compile for device <name>
-device-option                                       -- <key=value> ... Add option for the device mkspec
-direct2d                                            -- .......... Enable Direct2D support (auto) (Windows only)
-directfb                                            -- .......... Enable DirectFB support (no) (Unix only)
-docdir                                              -- <dir> ......... Documentation (DATADIR/doc)
-doubleconversion                                    -- .... Select used double conversion library (system/qt/no)
-egl                                                 -- ................. Enable EGL support (auto)
-eglfs                                               -- ............. Enable EGLFS support (auto; no on Android and Windows)
-evdev                                               -- ............. Enable evdev support (auto)
-eventfd                                             -- ............. Enable eventfd support
-evr                                                 -- ................. Enables EVR in DirectShow and WMF (auto)
-examplesdir                                         -- <dir> .... Examples (PREFIX/examples)
-external-hostbindir                                 -- <path> ... Path to Qt tools built for this machine.
-extprefix                                           -- <dir> ... The installation directory, as seen on the host machine.
-F                                                   -- <string> .......... Pass additional framework path (Apple only)
-feature-<feature>                                   -- ... Enable <feature>
-fontconfig                                          -- .......... Enable Fontconfig support (auto) (Unix only)
-force-asserts                                       -- ....... Enable Q_ASSERT even in release builds (no)
-force-debug-info                                    -- .... Create symbol files for release builds (no)
-framework                                           -- ........... Build Qt framework bundles (yes) (Apple only)
-freetype                                            -- ............ Select used FreeType (system/qt/no)
-gbm                                                 -- ............... Enable backends for GBM (auto) (Linux only)
-gc-binaries                                         -- ......... Place each function or data item into its own section
-gcc-sysroot                                         -- ......... With -sysroot, pass --sysroot to the compiler (yes)
-gcov                                                -- ................ Instrument with the GCov code coverage tool (no)
-gdb-index                                           -- ........... Index the debug info to speed up GDB
-gif                                                 -- ............... Enable reading support for GIF (auto)
-glib                                                -- ................ Enable Glib support (no; auto on Unix)
-gstreamer                                           -- (version) . Enable GStreamer support (auto)
-gtk                                                 -- ................. Enable GTK platform theme support (auto)
-gui                                                 -- ................. Build the Qt GUI module and dependencies (yes)
--gui                                                -- Values are listed in the order they are tried if not specified;
-harfbuzz                                            -- ............ Select used HarfBuzz-NG (system/qt/no)
-headerdir                                           -- <dir> ...... Header files (PREFIX/include)
-help                                            -h  -- ............ Display this help screen
-hostbindir                                          -- <dir> ..... Host executables (HOSTPREFIX/bin)
-hostdatadir                                         -- <dir> .... Data used by qmake (HOSTPREFIX)
-hostlibdir                                          -- <dir> ..... Host libraries (HOSTPREFIX/lib)
-hostprefix                                          -- (dir) .. The installation directory for build tools running on
-I                                                   -- <string> .......... Pass additional include path
-ico                                                 -- ............... Enable support for ICO (yes)
-iconv                                               -- ............... Enable iconv(3) support (posix/sun/gnu/no) (Unix only)
-icu                                                 -- ................. Enable ICU support (auto)
-imf                                                 -- ............... Enable IMF support (auto) (QNX only)
-importdir                                           -- <dir> ...... QML1 imports (ARCHDATADIR/imports)
-incredibuild-xge                                    -- .... Use the IncrediBuild XGE (no) (Windows only)
-inotify                                             -- ............. Enable inotify support
-jasper                                              -- .............. Enable JPEG-2000 support using the JasPer library (no)
-journald                                            -- .......... Enable journald support (no) (Unix only)
-kms                                                 -- ............... Enable backends for KMS (auto) (Linux only)
-L                                                   -- <string> .......... Pass additional library path
-lgmon                                               -- ............... Enable lgmon support (auto) (QNX only)
-libdir                                              -- <dir> ......... Libraries (PREFIX/lib)
-libexecdir                                          -- <dir> ..... Helper programs (ARCHDATADIR/bin on Windows,
-libinput                                            -- .......... Enable libinput support (auto)
-libjpeg                                             -- ........... Select used libjpeg (system/qt/no)
-libpng                                              -- ............ Select used libpng (system/qt/no)
-libproxy                                            -- ............ Enable use of libproxy (no)
-libudev............                                 -- Enable udev support (auto)
-linuxfb                                             -- ........... Enable Linux Framebuffer support (auto) (Linux only)
-list-features                                       -- ....... List available features. Note that some features
-list-libraries                                      -- ...... List possible external dependencies.
-ltcg                                                -- ................ Use Link Time Code Generation (no)
-make                                                -- <part> ......... Add <part> to the list of parts to be built.
-make-tool                                           -- <tool> .... Use <tool> to build qmake (nmake) (Windows only)
-mediaplayer-backend                                 -- <name> ... Select media player backend (Windows only)
-mips_dsp/-mips_dspr2                                -- Use MIPS DSP/rev2 instructions (auto)
-mirclient                                           -- ......... Enable Mir client support (no) (Linux only)
-mng                                                 -- ................. Enable MNG support (no)
-mp                                                  -- .................. Use multiple processors for compilation (MSVC only)
-mtdev                                               -- ............. Enable mtdev support (auto)
-no-dbus                                             -- ............. Do not build the Qt D-Bus module
-no-feature-<feature>                                -- Disable <feature> (none)
-no-gstreamer                                        -- ........ Disable support for GStreamer
-nomake                                              -- <part> ....... Exclude <part> from the list of parts to be built.
-no-opengl                                           -- ........... Disable OpenGL support
-no-openssl                                          -- .......... Do not use OpenSSL (default on Apple and WinRT)
-opengl                                              -- <api> ........ Enable OpenGL support. Supported APIs-
-opengles3                                           -- ........... Enable OpenGL ES 3.x support instead of ES 2.x (auto)
-opensource                                          -- .......... Build the Open-Source Edition of Qt
-openssl-linked                                      -- ...... Use OpenSSL and link to libssl (no)
-openssl-runtime                                     -- ..... Use OpenSSL and dynamically load libssl (auto)
-optimize-debug                                      -- ...... Enable debug-friendly optimizations in debug builds
-optimized-tools                                     -- ..... Build optimized host tools even in debug build (no)
-optimize-size                                       -- ....... Optimize release builds for size instead of speed (no)
-pch                                                 -- ................. Use precompiled headers (auto)
-pcre                                                -- ................ Select used libpcre2 (system/qt)
-pkg-config                                          -- .......... Use pkg-config (auto) (Unix only)
-platform                                            -- <target> ... Select host mkspec (detected)
-plugindir                                           -- <dir> ...... Plugins (ARCHDATADIR/plugins)
-plugin-manifests                                    -- .... Embed manifests into plugins (no) (Windows only)
-pps                                                 -- ................. Enable PPS support (auto) (QNX only)
-prefix                                              -- <dir> ...... The deployment directory, as seen on the target device.
-pulseaudio                                          -- .......... Enable PulseAudio support (auto) (Unix only)
-qmldir                                              -- <dir> ......... QML2 imports (ARCHDATADIR/qml)
-qpa                                                 -- <name> .......... Select default QPA backend(s) (e.g., xcb, cocoa, windows)
-qreal                                               -- <type> ........ typedef qreal to the specified type. (double)
-qt3d-animation.......                               -- Enable the Qt3D Animation aspect (yes)
-qt3d-extras                                         -- ......... Enable the Qt3D Extras aspect (yes)
-qt3d-input                                          -- .......... Enable the Qt3D Input aspect (yes)
-qt3d-logic                                          -- .......... Enable the Qt3D Logic aspect (yes)
-qt3d-profile-gl                                     -- ..... Enable OpenGL profiling (no)
-qt3d-profile-jobs                                   -- ... Enable jobs profiling (no)
-qt3d-render                                         -- ......... Enable the Qt3D Render aspect (yes)
-qt3d-simd                                           -- ........... Select level of SIMD support (no/sse2/avx2)
-qtlibinfix                                          -- <infix> .. Rename all libQt5*.so to libQt5*<infix>.so.
-qtnamespace                                         -- <name> .. Wrap all Qt library code in 'namespace <name> {...}'.
-R                                                   -- <string> .......... Add an explicit runtime library path to the Qt
-recheck                                             -- (test,...) .. Discard cached negative configure test results.
-recheck-all                                         -- ......... Discard all cached configure test results.
-redo                                                -- ................ Re-configure with previously used options.
-reduce-exports                                      -- ...... Reduce amount of exported symbols (auto)
-reduce-relocations                                  -- .. Reduce amount of relocations (auto) (Unix only)
-release                                             -- ............. Build Qt with debugging turned off (yes)
-rpath                                               -- ............... Link Qt libraries and executables using the library
-sanitize                                            -- {address|thread|memory|undefined}
-sctp                                                -- ................ Enable SCTP support (no)
-sdk                                                 -- <sdk> ........... Build Qt using Apple provided SDK <sdk>. The argument
-securetransport                                     -- ..... Use SecureTransport (auto) (Apple only)
-separate-debug-info                                 -- . Split off debug information to separate files (no)
-shared                                              -- .............. Build shared Qt libraries (yes) (no for UIKit)
-silent                                              -- .............. Reduce the build output so that warnings and errors
-skip                                                -- <repo> ......... Exclude an entire repository from the build.
-slog2                                               -- ............. Enable slog2 support (auto) (QNX only)
-sql-<driver>                                        -- ........ Enable SQL <driver> plugin. Supported drivers-
-sqlite                                              -- .............. Select used sqlite3 (system/qt)
-sse2                                                -- ................ Use SSE2 instructions (auto)
-sse3/-ssse3/-sse4.1/-sse4.2/-avx/-avx2/-avx512      -- Enable use of particular x86 instructions (auto)
-ssl                                                 -- ................. Enable either SSL support method (auto)
-static                                              -- .............. Build static Qt libraries (no) (yes for UIKit)
-static-runtime                                      -- ...... With -static, use static runtime (no) (Windows only)
-strip                                               -- ............... Strip release binaries of unneeded symbols (yes)
-sysconfdir                                          -- <dir> ..... Settings used by Qt programs (PREFIX/etc/xdg)
-syslog                                              -- ............ Enable syslog support (no) (Unix only)
-sysroot                                             -- <dir> ....... Set <dir> as the target sysroot
-system-proxies                                      -- ...... Use system network proxies by default (yes)
-testcocoon                                          -- .......... Instrument with the TestCocoon code coverage tool (no)
-testsdir                                            -- <dir> ....... Tests (PREFIX/tests)
-tiff                                                -- ................ Enable TIFF support (system/qt/no)
-trace                                               -- (backend) ..... Enable instrumentation with tracepoints.
-translationdir                                      -- <dir> . Translations (DATADIR/translations)
-tslib                                               -- ............. Enable tslib support (auto)
-use-gold-linker                                     -- ..... Use the GNU gold linker (auto)
-verbose                                         -v  -- ......... Print verbose messages during configuration
-warnings-are-errors                                 -- . Treat warnings as errors (no; yes if -developer-build)
-webengine-alsa                                      -- ................ Enable ALSA support (auto) (Linux only)
-webengine-embedded-build                            -- ...... Enable Linux embedded build (auto)
-webengine-ffmpeg                                    -- .............. Use system FFmpeg libraries (system/qt)
-webengine-icu                                       -- ................. Use system ICU libraries (system/qt)
-webengine-native-spellchecker                       -- . Enable support for native spellchecker (no)
-webengine-opus                                      -- ................ Use system Opus libraries (system/qt)
-webengine-pepper-plugins                            -- ...... Enable use of Pepper Flash and Widevine
-webengine-printing-and-pdf                          -- .... Enable use of printing and output to PDF
-webengine-proprietary-codecs                        -- .. Enable support for proprietary codecs (no)
-webengine-pulseaudio                                -- .......... Enable PulseAudio support (auto)
-webengine-spellchecker                              -- ........ Enable support for spellchecker (yes)
-webengine-webp                                      -- ................ Use system WebP libraries (system/qt)
-webengine-webrtc                                    -- .............. Enable support for WebRTC (auto)
-webp                                                -- ................ Enable WEBP support (system/qt/no)
-widgets                                             -- ............. Build the Qt Widgets module and dependencies (yes)
-xcb                                                 -- ............... Enable X11 support. Select used xcb-* libraries (system/qt/no)
-xcb-xinput                                          -- ........ Enable XInput2 support (auto)
-xcb-xlib.............                               -- Enable Xcb-Xlib support (auto)
-xkbcommon                                           -- ......... Enable key mapping support (auto)
-xplatform                                           -- <target> .. Select target mkspec when cross-compiling (PLATFORM)
-zlib                                                -- ................ Select used zlib (system/qt)
-developer-build
```

```
./configure -list-libraries 
+ cd qtbase
+ /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/configure -top-level -list-libraries
Creating qmake...
............................................................................................Done.
Info: creating super cache file /home/wh/Downloads/qt-everywhere-src-5.12.9/.qmake.super
Info: creating cache file /home/wh/Downloads/qt-everywhere-src-5.12.9/.qmake.cache

qtbase:
  dbus
  libudev
  zlib
core:
  doubleconversion
  glib
  iconv
  icu
  journald
  libatomic
  libdl
  librt
  lttng_ust
  pcre2
  pps
network:
  libproxy
  network
  openssl
gui:
  atspi
  d2d1
  d2d1_1
  d3d11
  d3d11_1
  d3d9
  d3dcompiler
  directfb
  drm
  dwrite
  dwrite_1
  dwrite_2
  dxgi
  dxgi1_2
  dxguid
  egl
  fontconfig
  freetype
  gbm
  harfbuzz
  integrityhid
  lgmon
  libinput
  libjpeg
  libpng
  mirclient
  mtdev
  opengl
  opengl_es2
  openvg
  tslib
  v4l2
  vulkan
  wayland_server
  x11sm
  xcb
  xcb_glx
  xcb_icccm
  xcb_image
  xcb_keysyms
  xcb_randr
  xcb_render
  xcb_renderutil
  xcb_shape
  xcb_shm
  xcb_sync
  xcb_xfixes
  xcb_xinerama
  xcb_xinput
  xcb_xkb
  xcb_xlib
  xkbcommon
  xkbcommon_x11
  xlib
  xrender
widgets:
  gtk3
printsupport:
  cups
sqldrivers:
  db2
  ibase
  mysql
  oci
  odbc
  psql
  sqlite
  sqlite2
  tds
imageformats:
  jasper
  mng
  tiff
  webp
gamepad:
  sdl2
3dcore:
  assimp
geometryloaders:
  fbx
waylandclient:
  glx
  wayland_client
  wayland_cursor
  wayland_egl
  xcomposite
waylandcompositor:
  glx
  wayland_egl
  wayland_kms
  wayland_server
  xcomposite
bluetooth:
  bluez
sensors:
  sensorfw
positioning:
  gypsy
multimedia:
  alsa
  avfoundation
  directshow
  gstreamer
  gstreamer_app
  gstreamer_photography
  libresourceqt5
  mmrenderer
  openal
  pulseaudio
  wmf
webenginecore:
  pulseaudio
  webengine_dbus
  webengine_ffmpeg
  webengine_fontconfig
  webengine_freetype
  webengine_glib
  webengine_harfbuzz
  webengine_icu
  webengine_jpeglib
  webengine_jsoncpp
  webengine_lcms2
  webengine_libdrm
  webengine_libevent
  webengine_libxml2
  webengine_minizip
  webengine_nss
  webengine_opus
  webengine_png
  webengine_poppler_cpp
  webengine_protobuf
  webengine_webp
  webengine_x11
  webengine_xcomposite
  webengine_xcursor
  webengine_xi
  webengine_xtst
  webengine_zlib
```
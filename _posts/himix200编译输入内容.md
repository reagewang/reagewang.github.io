qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++/qmake.conf
增加了以下两行的
QMAKE_CFLAGS_RELEASE += -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4
QMAKE_CXXFLAGS_RELEASE += -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive
```
+ cd qtbase
+ /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/configure -top-level -prefix /opt/qt5.12.9_hi3516dv300 -xplatform linux-arm-himix200-linux-g++ -v -opensource -confirm-license -release -silent -strip -no-rpath -no-pch -no-pkg-config -make libs -nomake examples -nomake tests -nomake tools -no-iconv -qt-zlib -qt-pcre -no-openssl -qt-freetype -no-opengl -linuxfb -qt-libpng -qt-libjpeg -qt-sqlite -no-assimp -skip qtserialbus
Performing shadow build...
Preparing build tree...
Creating qmake...
make: Nothing to be done for 'first'.
Command line: -prefix /opt/qt5.12.9_hi3516dv300 -xplatform linux-arm-himix200-linux-g++ -v -opensource -confirm-license -release -silent -strip -no-rpath -no-pch -no-pkg-config -make libs -nomake examples -nomake tests -nomake tools -no-iconv -qt-zlib -qt-pcre -no-openssl -qt-freetype -no-opengl -linuxfb -qt-libpng -qt-libjpeg -qt-sqlite -no-assimp -skip qtserialbus

This is the Qt Open Source Edition.

You have already accepted the terms of the Open Source license.

Running configuration tests...

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests && /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -pipe -fuse-ld=gold -o conftest-out conftest.cpp
> collect2: fatal error: cannot find 'ld'
> compilation terminated.

+ /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -dumpmachine
> arm-linux-gnueabi

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/verifyspec && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/verifyspec
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/verifyspec && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/verifyspec -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o verifyspec.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/verifyspec/verifyspec.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o verifyspec verifyspec.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/arch && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/arch && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o arch.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch/arch.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o arch arch.o      
Detected architecture: arm (neon)

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/arch && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch/arch_host.pro
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/arch && MAKEFLAGS= /usr/bin/make clean && MAKEFLAGS= /usr/bin/make
> rm -f arch.o
> rm -f *~ core *.core
> g++ -c -pipe -O2 -std=gnu++11 -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-g++ -o arch.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/arch/arch.cpp
> g++ -Wl,-O1 -o arch_host arch.o      
Detected architecture: x86_64 (mmx sse sse2)

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/alloca_h && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/alloca_h
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/alloca_h && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o alloca_h main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++14 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++14
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++14 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -std=gnu++1y -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o c++14 main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++1z && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++1z
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c++1z && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -std=gnu++1z -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> main.cpp:8:19: fatal error: variant: No such file or directory
>  #include <variant>
>                    ^
> compilation terminated.
> Makefile:167: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c99 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c99
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c99 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-gcc -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -std=gnu99 -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.c
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o c99 main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c11 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c11
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/c11 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-gcc -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -std=gnu11 -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.c
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o c11 main.o      
Global lib dirs: [] [/opt/hisi-linux/x86-arm/arm-himix200-linux/lib/gcc/arm-linux-gnueabi/6.3.0 /opt/hisi-linux/x86-arm/arm-himix200-linux/lib/gcc /opt/hisi-linux/x86-arm/arm-himix200-linux/arm-linux-gnueabi/lib /opt/hisi-linux/x86-arm/arm-himix200-linux/target/lib /opt/hisi-linux/x86-arm/arm-himix200-linux/target/usr/lib]
Global inc dirs: [] [/opt/hisi-linux/x86-arm/arm-himix200-linux/arm-linux-gnueabi/include/c++/6.3.0 /opt/hisi-linux/x86-arm/arm-himix200-linux/arm-linux-gnueabi/include/c++/6.3.0/arm-linux-gnueabi /opt/hisi-linux/x86-arm/arm-himix200-linux/arm-linux-gnueabi/include/c++/6.3.0/backward /opt/hisi-linux/x86-arm/arm-himix200-linux/lib/gcc/arm-linux-gnueabi/6.3.0/include /opt/hisi-linux/x86-arm/arm-himix200-linux/lib/gcc/arm-linux-gnueabi/6.3.0/include-fixed /opt/hisi-linux/x86-arm/arm-himix200-linux/arm-linux-gnueabi/include /opt/hisi-linux/x86-arm/arm-himix200-linux/target/usr/include]

Trying source 0 (type pkgConfig) of library dbus ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library dbus ...
  => source failed condition 'config.win32'.
Trying source 2 (type inline) of library dbus ...
None of [libdbus-1.so libdbus-1.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests && /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -pipe -Wl,--enable-new-dtags -o conftest-out conftest.cpp

Trying source 0 (type pkgConfig) of library host_dbus ...
+ /usr/bin/pkg-config --exists --silence-errors dbus-1 '>=' 1.2
pkg-config did not find package.
  => source produced no result.
Trying source 1 (type inline) of library host_dbus ...
 => source accepted.

Trying source 0 (type pkgConfig) of library libudev ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library libudev ...
None of [libudev.so libudev.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/posix_fallocate && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/posix_fallocate
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/posix_fallocate && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o posix_fallocate main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/x86_simd && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" CONFIG+=add_cflags DEFINES+=NO_ATTRIBUTE SIMD=rdrnd /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/x86_simd
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/x86_simd && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -mrdrnd -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC -DNO_ATTRIBUTE -DQT_COMPILER_SUPPORTS_RDRND -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/x86_simd -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/x86_simd/main.cpp
> arm-himix200-linux-g++: error: unrecognized command line option '-mrdrnd'
> Makefile:170: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_exports && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_exports
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_exports && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -fvisibility=hidden -fvisibility-inlines-hidden -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> rm -f libreduce_exports.so.1.0.0 libreduce_exports.so libreduce_exports.so.1 libreduce_exports.so.1.0
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -shared -Wl,-soname,libreduce_exports.so.1 -o libreduce_exports.so.1.0.0 main.o      
> ln -s libreduce_exports.so.1.0.0 libreduce_exports.so
> ln -s libreduce_exports.so.1.0.0 libreduce_exports.so.1
> ln -s libreduce_exports.so.1.0.0 libreduce_exports.so.1.0

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_relocations && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_relocations
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/reduce_relocations && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> main.cpp:3:4: error: #error Symbolic function binding on this architecture may be broken, disabling it (see QTBUG-36129).
>  #  error Symbolic function binding on this architecture may be broken, disabling it (see QTBUG-36129).
>     ^~~~~
> Makefile:187: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/stl && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/stl
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/stl && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/stl -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o stltest.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/config.tests/stl/stltest.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o stl stltest.o      

Trying source 0 (type inline) of library librt ...
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/librt && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += librt' 'QMAKE_LIBS_LIBRT = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/librt
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/librt && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o librt main.o      
 => source accepted.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/clock-monotonic && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += librt' 'QMAKE_LIBS_LIBRT = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/clock-monotonic
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/clock-monotonic && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o clock-monotonic main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_future && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_future
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_future && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o cxx11_future main.o   -lpthread   
> main.o: In function `std::__future_base::_Deferred_state<std::_Bind_simple<main::{lambda()#1} ()>, int>::_M_complete_async()':
> main.cpp:(.text+0x39c): undefined reference to `std::__atomic_futex_unsigned_base::_M_futex_notify_all(unsigned int*)'
> main.o: In function `std::thread::_State_impl<std::_Bind_simple<std::__future_base::_Async_state_impl<std::_Bind_simple<main::{lambda()#1} ()>, int>::_Async_state_impl(main::{lambda()#1} (&&)())::{lambda()#1} ()> >::_M_run()':
> main.cpp:(.text+0xab0): undefined reference to `std::__atomic_futex_unsigned_base::_M_futex_notify_all(unsigned int*)'
> main.o: In function `std::__future_base::_State_baseV2::_M_break_promise(std::unique_ptr<std::__future_base::_Result_base, std::__future_base::_Result_base::_Deleter>)':
> main.cpp:(.text._ZNSt13__future_base13_State_baseV216_M_break_promiseESt10unique_ptrINS_12_Result_baseENS2_8_DeleterEE[_ZNSt13__future_base13_State_baseV216_M_break_promiseESt10unique_ptrINS_12_Result_baseENS2_8_DeleterEE]+0x1e0): undefined reference to `std::__atomic_futex_unsigned_base::_M_futex_notify_all(unsigned int*)'
> main.o: In function `main':
> main.cpp:(.text.startup+0x1f8): undefined reference to `std::__atomic_futex_unsigned_base::_M_futex_wait_until(unsigned int*, unsigned int, bool, std::chrono::duration<long long, std::ratio<1ll, 1ll> >, std::chrono::duration<long long, std::ratio<1ll, 1000000000ll> >)'
> collect2: error: ld returned 1 exit status
> Makefile:67: recipe for target 'cxx11_future' failed
> make: *** [cxx11_future] Error 1

Trying source 0 (type inline) of library libdl ...
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += libdl' 'QMAKE_LIBS_LIBDL = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o libdl main.o      
> main.o: In function `main':
> main.cpp:(.text.startup+0xc): undefined reference to `dlopen'
> main.cpp:(.text.startup+0x10): undefined reference to `dlclose'
> main.cpp:(.text.startup+0x1c): undefined reference to `dlsym'
> main.cpp:(.text.startup+0x20): undefined reference to `dlerror'
> collect2: error: ld returned 1 exit status
> Makefile:67: recipe for target 'libdl' failed
> make: *** [libdl] Error 1
 => source failed verification.
Trying source 1 (type inline) of library libdl ...
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += libdl' 'QMAKE_LIBS_LIBDL = /opt/hisi-linux/x86-arm/arm-himix200-linux/target/usr/lib/libdl.so' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libdl && MAKEFLAGS= /usr/bin/make clean && MAKEFLAGS= /usr/bin/make
> rm -f main.o
> rm -f *~ core *.core
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o libdl main.o   /opt/hisi-linux/x86-arm/arm-himix200-linux/target/usr/lib/libdl.so   
 => source accepted.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/eventfd && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/eventfd
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/eventfd && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o eventfd main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/futimens && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/futimens
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/futimens && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -Wall -W -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o futimens main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getauxval && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getauxval
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getauxval && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o getauxval main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getentropy && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getentropy
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getentropy && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o getentropy main.o      

Trying source 0 (type pkgConfig) of library glib ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/glibc && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/glibc
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/glibc && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o glibc main.o      

Trying source 0 (type inline) of library icu ...
  => source failed condition 'config.win32 && !features.shared'.
Trying source 1 (type inline) of library icu ...
  => source failed condition 'config.win32 && features.shared'.
Trying source 2 (type inline) of library icu ...
None of [libicui18n.so libicui18n.a] found in [] and global paths.
None of [libicuuc.so libicuuc.a] found in [] and global paths.
None of [libicudata.so libicudata.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/inotify && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/inotify
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/inotify && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o inotify main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipc_sysv && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipc_sysv
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipc_sysv && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o ipc_sysv main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linkat && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linkat
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linkat && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o linkat main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ppoll && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ppoll
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ppoll && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o ppoll main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/renameat2 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/renameat2
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/renameat2 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> main.cpp: In function 'int main(int, char**)':
> main.cpp:9:53: error: 'RENAME_NOREPLACE' was not declared in this scope
>      renameat2(AT_FDCWD, argv[1], AT_FDCWD, argv[2], RENAME_NOREPLACE | RENAME_WHITEOUT);
>                                                      ^~~~~~~~~~~~~~~~
> main.cpp:9:72: error: 'RENAME_WHITEOUT' was not declared in this scope
>      renameat2(AT_FDCWD, argv[1], AT_FDCWD, argv[2], RENAME_NOREPLACE | RENAME_WHITEOUT);
>                                                                         ^~~~~~~~~~~~~~~
> main.cpp:9:87: error: 'renameat2' was not declared in this scope
>      renameat2(AT_FDCWD, argv[1], AT_FDCWD, argv[2], RENAME_NOREPLACE | RENAME_WHITEOUT);
>                                                                                        ^
> Makefile:169: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

Trying source 0 (type inline) of library slog2 ...
None of [libslog2.so libslog2.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/statx && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/statx
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/statx && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> main.cpp: In function 'int main(int, char**)':
> main.cpp:11:18: error: aggregate 'main(int, char**)::statx statxbuf' has incomplete type and cannot be defined
>      struct statx statxbuf;
>                   ^~~~~~~~
> main.cpp:12:25: error: 'STATX_BASIC_STATS' was not declared in this scope
>      unsigned int mask = STATX_BASIC_STATS;
>                          ^~~~~~~~~~~~~~~~~
> main.cpp:13:32: error: 'AT_STATX_SYNC_AS_STAT' was not declared in this scope
>      return statx(AT_FDCWD, "", AT_STATX_SYNC_AS_STAT, mask, &statxbuf);
>                                 ^~~~~~~~~~~~~~~~~~~~~
> main.cpp:13:70: error: invalid use of incomplete type 'struct main(int, char**)::statx'
>      return statx(AT_FDCWD, "", AT_STATX_SYNC_AS_STAT, mask, &statxbuf);
>                                                                       ^
> main.cpp:11:12: note: forward declaration of 'struct main(int, char**)::statx'
>      struct statx statxbuf;
>             ^~~~~
> Makefile:169: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

Trying source 0 (type inline) of library libatomic ...
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libatomic && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += libatomic' 'QMAKE_LIBS_LIBATOMIC = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libatomic
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libatomic && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -std=gnu++11 -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o libatomic main.o      
 => source accepted.

Trying source 0 (type inline) of library doubleconversion ...
None of [libdouble-conversion.so libdouble-conversion.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cloexec && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cloexec
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cloexec && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o cloexec main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_random && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_random
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/cxx11_random && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o cxx11_random main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/atomicfptr && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/atomicfptr
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/atomicfptr && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -std=gnu++11 -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o atomicfptr main.o      
Trying source 0 (type makeSpec) of library network ...
 => source accepted.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getifaddrs && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += network' 'QMAKE_LIBS_NETWORK = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getifaddrs
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/getifaddrs && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o getifaddrs main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipv6ifname && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" 'QMAKE_USE += network' 'QMAKE_LIBS_NETWORK = ' /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipv6ifname
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/ipv6ifname && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o ipv6ifname main.o      

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linux-netlink && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linux-netlink
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linux-netlink && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> main.cpp: In function 'int main(int, char**)':
> main.cpp:18:67: error: 'IFA_F_MANAGETEMPADDR' was not declared in this scope
>      (void)(IFA_F_SECONDARY | IFA_F_DEPRECATED | IFA_F_PERMANENT | IFA_F_MANAGETEMPADDR);
>                                                                    ^~~~~~~~~~~~~~~~~~~~
> Makefile:169: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

Trying source 0 (type pkgConfig) of library xkbcommon ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library xkbcommon ...
None of [libxkbcommon.so libxkbcommon.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library drm ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library drm ...
None of [libdrm.so libdrm.a] found in [] and global paths.
  => source produced no result.
Trying source 2 (type inline) of library drm ...
  => source failed condition 'config.integrity'.

Trying source 0 (type pkgConfig) of library openvg ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type makeSpec) of library openvg ...
None of [libOpenVG.so libOpenVG.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/evdev && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/evdev
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/evdev && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o evdev main.o      

Trying source 0 (type pkgConfig) of library gbm ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linuxfb && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linuxfb
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linuxfb && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o linuxfb main.o      

Trying source 0 (type pkgConfig) of library mtdev ...
pkg-config use disabled globally.
  => source produced no result.


Trying source 0 (type inline) of library harfbuzz ...
None of [libharfbuzz.so libharfbuzz.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library tslib ...
None of [libts.so libts.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library vulkan ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type makeSpec) of library vulkan ...
vulkan/vulkan.h not found in [] and global paths.
  => source produced no result.

Trying source 0 (type makeSpec) of library xlib ...
None of [libXext.so libXext.a] found in [] and global paths.
None of [libX11.so libX11.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library cups ...
None of [libcups.so libcups.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library db2 ...
  => source failed condition 'config.win32'.
Trying source 1 (type inline) of library db2 ...
None of [libdb2.so libdb2.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library ibase ...
  => source failed condition 'config.win32'.
Trying source 1 (type inline) of library ibase ...
None of [libgds.so libgds.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type mysqlConfig) of library mysql ...
mysql_config not found.
  => source produced no result.
Trying source 1 (type mysqlConfig) of library mysql ...
mysql_config not found.
  => source produced no result.
Trying source 2 (type mysqlConfig) of library mysql ...
mysql_config not found.
  => source produced no result.
Trying source 3 (type mysqlConfig) of library mysql ...
mysql_config not found.
  => source produced no result.
Trying source 4 (type inline) of library mysql ...
None of [libmysqlclient_r.so libmysqlclient_r.a] found in [] and global paths.
  => source produced no result.
Trying source 5 (type inline) of library mysql ...
  => source failed condition 'config.win32'.
Trying source 6 (type inline) of library mysql ...
None of [libmysqlclient.so libmysqlclient.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library oci ...
  => source failed condition 'config.win32'.
Trying source 1 (type inline) of library oci ...
None of [libclntsh.so libclntsh.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library odbc ...
  => source failed condition 'config.win32'.
Trying source 1 (type inline) of library odbc ...
  => source failed condition 'config.darwin'.
Trying source 2 (type inline) of library odbc ...
None of [libodbc.so libodbc.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library psql ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type psqlConfig) of library psql ...
pg_config not found.
  => source produced no result.
Trying source 2 (type psqlEnv) of library psql ...
  => source failed condition 'config.win32'.
Trying source 3 (type psqlEnv) of library psql ...
None of [libpq.so libpq.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library sqlite2 ...
None of [libsqlite.so libsqlite.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type sybaseEnv) of library tds ...
  => source failed condition 'config.win32'.
Trying source 1 (type sybaseEnv) of library tds ...
None of [libsybdb.so libsybdb.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library jasper ...
None of [libjasper.so libjasper.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library mng ...
None of [libmng.so libmng.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library tiff ...
None of [libtiff.so libtiff.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type inline) of library webp ...
None of [libwebp.so libwebp.a] found in [] and global paths.
None of [libwebpdemux.so libwebpdemux.a] found in [] and global paths.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/d3d12 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtdeclarative/config.tests/d3d12
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/d3d12 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtdeclarative/config.tests/d3d12 -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o d3d12.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtdeclarative/config.tests/d3d12/d3d12.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtdeclarative/config.tests/d3d12/d3d12.cpp:40:19: fatal error: d3d12.h: No such file or directory
>  #include <d3d12.h>
>                    ^
> compilation terminated.
> Makefile:169: recipe for target 'd3d12.o' failed
> make: *** [d3d12.o] Error 1

Trying source 0 (type pkgConfig) of library sdl2 ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library sdl2 ...
  => source failed condition 'config.darwin'.
Trying source 2 (type inline) of library sdl2 ...
  => source failed condition 'config.win32'.
Trying source 3 (type inline) of library sdl2 ...
None of [libSDL2.so libSDL2.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type fbx) of library fbx ...
None of [libfbxsdk.so libfbxsdk.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library wayland-client ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library wayland-client ...
None of [libwayland-client.so libwayland-client.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library wayland-egl ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library wayland-egl ...
None of [libwayland-egl.so libwayland-egl.a] found in [] and global paths.
  => source produced no result.
Trying source 2 (type inline) of library wayland-egl ...
None of [libEGL.so libEGL.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library wayland-server ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library wayland-server ...
None of [libwayland-server.so libwayland-server.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library bluez ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library bluez ...
None of [libbluetooth.so libbluetooth.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library sensorfw ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library gypsy ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/winrt && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtlocation/config.tests/winrt
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/winrt && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtlocation/config.tests/winrt -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtlocation/config.tests/winrt/main.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtlocation/config.tests/winrt/main.cpp:30:28: fatal error: windows.system.h: No such file or directory
>  #include <windows.system.h>
>                             ^
> compilation terminated.
> Makefile:169: recipe for target 'main.o' failed
> make: *** [main.o] Error 1

Trying source 0 (type inline) of library alsa ...
None of [libasound.so libasound.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library gstreamer_1_0 ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library gstreamer_0_10 ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linux_v4l && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtmultimedia/config.tests/linux_v4l
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/linux_v4l && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtmultimedia/config.tests/linux_v4l -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o main.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtmultimedia/config.tests/linux_v4l/main.cpp
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -Wl,-O1 -o linux_v4l main.o      

Trying source 0 (type pkgConfig) of library openal ...
pkg-config use disabled globally.
  => source produced no result.
Trying source 1 (type inline) of library openal ...
  => source failed condition 'config.win32'.
Trying source 2 (type inline) of library openal ...
  => source failed condition 'config.darwin'.
Trying source 3 (type inline) of library openal ...
None of [libopenal.so libopenal.a] found in [] and global paths.
  => source produced no result.

Trying source 0 (type pkgConfig) of library pulseaudio ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library libresourceqt5 ...
pkg-config use disabled globally.
  => source produced no result.


+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/alsa && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/alsa
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/alsa && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/alsa -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o alsatest.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/alsa/alsatest.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/alsa/alsatest.cpp:29:28: fatal error: alsa/asoundlib.h: No such file or directory
>  #include <alsa/asoundlib.h>
>                             ^
> compilation terminated.
> Makefile:169: recipe for target 'alsatest.o' failed
> make: *** [alsatest.o] Error 1



Required bison could not be found.

Required flex could not be found.

Required gperf could not be found.

Found host pkg-config: /usr/bin/pkg-config

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests && /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -pipe -Wl,-z,noexecstack -o conftest-out conftest.cpp

Trying source 0 (type pkgConfig) of library webengine-x11 ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-poppler-cpp ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library pulseaudio ...
pkg-config use disabled globally.
  => source produced no result.


Trying source 0 (type pkgConfig) of library webengine-dbus ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-fontconfig ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-glib ...
pkg-config use disabled globally.
  => source produced no result.

Found ldd from path: /usr/bin/ldd
+ /usr/bin/ldd --version
> ldd (Ubuntu GLIBC 2.23-0ubuntu11.2) 2.23
> Copyright (C) 2016 Free Software Foundation, Inc.
> This is free software; see the source for copying conditions.  There is NO
> warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
> Written by Roland McGrath and Ulrich Drepper.
Found libc version: 2.23

Trying source 0 (type pkgConfig) of library webengine-jsoncpp ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/khr && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/khr
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/khr && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/khr -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o khr.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/khr/khr.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/khr/khr.cpp:29:29: fatal error: KHR/khrplatform.h: No such file or directory
>  #include <KHR/khrplatform.h>
>                              ^
> compilation terminated.
> Makefile:169: recipe for target 'khr.o' failed
> make: *** [khr.o] Error 1

Trying source 0 (type pkgConfig) of library webengine-libdrm ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-libevent ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/libvpx && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/libvpx
> Project WARNING: Cross compiling without sysroot. Disabling pkg-config.
> Project WARNING: Cross compiling without sysroot. Disabling pkg-config.
> sh: 1: --exists: not found
> Project ERROR: vpx development package not found

Trying source 0 (type pkgConfig) of library webengine-webp ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-libxml2 ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-minizip ...
pkg-config use disabled globally.
  => source produced no result.

Building own ninja

Trying source 0 (type pkgConfig) of library webengine-nss ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-opus ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-protobuf ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/re2 && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/re2
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/re2 && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/re2 -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o re2.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/re2/re2.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/re2/re2.cpp:29:21: fatal error: re2/re2.h: No such file or directory
>  #include <re2/re2.h>
>                      ^
> compilation terminated.
> Makefile:169: recipe for target 're2.o' failed
> make: *** [re2.o] Error 1

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/snappy && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/snappy
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/snappy && MAKEFLAGS= /usr/bin/make
> /opt/hisi-linux/x86-arm/arm-himix200-linux/bin/arm-himix200-linux-g++ -c -pipe -O2 -mcpu=cortex-a7 -mfloat-abi=softfp -mfpu=neon-vfpv4 -fpermissive -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/snappy -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-arm-himix200-linux-g++ -o snappy.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/snappy/snappy.cpp
> /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/snappy/snappy.cpp:29:20: fatal error: snappy.h: No such file or directory
>  #include <snappy.h>
>                     ^
> compilation terminated.
> Makefile:169: recipe for target 'snappy.o' failed
> make: *** [snappy.o] Error 1

Trying source 0 (type pkgConfig) of library webengine-xcomposite ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-xcursor ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-xi ...
pkg-config use disabled globally.
  => source produced no result.

Trying source 0 (type pkgConfig) of library webengine-xtst ...
pkg-config use disabled globally.
  => source produced no result.

+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/hostcompiler && /home/wh/Downloads/qt-src-5.12.9-build-himix200/qtbase/bin/qmake "CONFIG -= qt debug_and_release app_bundle lib_bundle" "CONFIG += shared warn_off console single_arch" -early "CONFIG += cross_compile" /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/hostcompiler
+ cd /home/wh/Downloads/qt-src-5.12.9-build-himix200/config.tests/hostcompiler && MAKEFLAGS= /usr/bin/make
> g++ -c -pipe -m32 -O2 -std=gnu++11 -w -fPIC  -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/hostcompiler -I. -I/home/wh/Downloads/qt-everywhere-src-5.12.9/qtbase/mkspecs/linux-g++ -o main.o /home/wh/Downloads/qt-everywhere-src-5.12.9/qtwebengine/config.tests/hostcompiler/main.cpp
> g++ -m32 -Wl,-O1 -o hostcompiler main.o      
> /usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/5/libstdc++.so when searching for -lstdc++
> /usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/5/libstdc++.a when searching for -lstdc++
> /usr/bin/ld: cannot find -lstdc++
> collect2: error: ld returned 1 exit status
> Makefile:67: recipe for target 'hostcompiler' failed
> make: *** [hostcompiler] Error 1
Done running configuration tests.

Configure summary:

Building on: linux-g++ (x86_64, CPU features: mmx sse sse2)
Building for: linux-arm-himix200-linux-g++ (arm, CPU features: neon)
Target compiler: gcc 6.3.0
Configuration: cross_compile compile_examples enable_new_dtags largefile neon silent shared release c++11 c++14 concurrent dbus no-pkg-config reduce_exports stl
Build options:
  Mode ................................... release
  Optimize release build for size ........ no
  Building shared libraries .............. yes
  Using C standard ....................... C11
  Using C++ standard ..................... C++14
  Using ccache ........................... no
  Using gold linker ...................... no
  Using new DTAGS ........................ yes
  Using precompiled headers .............. no
  Using LTCG ............................. no
  Target compiler supports:
    NEON ................................. yes
  Build parts ............................ libs
Qt modules and options:
  Qt Concurrent .......................... yes
  Qt D-Bus ............................... yes
  Qt D-Bus directly linked to libdbus .... no
  Qt Gui ................................. yes
  Qt Network ............................. yes
  Qt Sql ................................. yes
  Qt Testlib ............................. yes
  Qt Widgets ............................. yes
  Qt Xml ................................. yes
Support enabled for:
  Using pkg-config ....................... no
  udev ................................... no
  Using system zlib ...................... no
Qt Core:
  DoubleConversion ....................... yes
    Using system DoubleConversion ........ no
  GLib ................................... no
  iconv .................................. no
  ICU .................................... no
  Tracing backend ........................ <none>
  Logging backends:
    journald ............................. no
    syslog ............................... no
    slog2 ................................ no
  Using system PCRE2 ..................... no
Qt Network:
  getifaddrs() ........................... yes
  IPv6 ifname ............................ yes
  libproxy ............................... no
  Linux AF_NETLINK ....................... no
  OpenSSL ................................ no
    Qt directly linked to OpenSSL ........ no
  OpenSSL 1.1 ............................ no
  DTLS ................................... no
  SCTP ................................... no
  Use system proxies ..................... yes
Qt Gui:
  Accessibility .......................... yes
  FreeType ............................... yes
    Using system FreeType ................ no
  HarfBuzz ............................... yes
    Using system HarfBuzz ................ no
  Fontconfig ............................. no
  Image formats:
    GIF .................................. yes
    ICO .................................. yes
    JPEG ................................. yes
      Using system libjpeg ............... no
    PNG .................................. yes
      Using system libpng ................ no
  EGL .................................... no
  OpenVG ................................. no
  OpenGL:
    Desktop OpenGL ....................... no
    OpenGL ES 2.0 ........................ no
    OpenGL ES 3.0 ........................ no
    OpenGL ES 3.1 ........................ no
    OpenGL ES 3.2 ........................ no
  Vulkan ................................. no
  Session Management ..................... yes
Features used by QPA backends:
  evdev .................................. yes
  libinput ............................... no
  INTEGRITY HID .......................... no
  mtdev .................................. no
  tslib .................................. no
  xkbcommon .............................. no
  X11 specific:
    XLib ................................. no
    XCB Xlib ............................. no
    EGL on X11 ........................... no
QPA backends:
  DirectFB ............................... no
  EGLFS .................................. no
  LinuxFB ................................ yes
  VNC .................................... yes
  Mir client ............................. no
Qt Sql:
  SQL item models ........................ yes
Qt Widgets:
  GTK+ ................................... no
  Styles ................................. Fusion Windows
Qt PrintSupport:
  CUPS ................................... no
Qt Sql Drivers:
  DB2 (IBM) .............................. no
  InterBase .............................. no
  MySql .................................. no
  OCI (Oracle) ........................... no
  ODBC ................................... no
  PostgreSQL ............................. no
  SQLite2 ................................ no
  SQLite ................................. yes
    Using system provided SQLite ......... no
  TDS (Sybase) ........................... no
Qt Testlib:
  Tester for item models ................. yes
Further Image Formats:
  JasPer ................................. no
  MNG .................................... no
  TIFF ................................... yes
    Using system libtiff ................. no
  WEBP ................................... yes
    Using system libwebp ................. no
Qt QML:
  QML network support .................... yes
  QML debugging and profiling support .... yes
  QML sequence object .................... yes
  QML list model ......................... yes
  QML XML http request ................... yes
  QML Locale ............................. yes
  QML delegate model ..................... yes
Qt Quick:
  Direct3D 12 ............................ no
  AnimatedImage item ..................... yes
  Canvas item ............................ yes
  Support for Qt Quick Designer .......... yes
  Flipable item .......................... yes
  GridView item .......................... yes
  ListView item .......................... yes
  TableView item ......................... yes
  Path support ........................... yes
  PathView item .......................... yes
  Positioner items ....................... yes
  Repeater item .......................... yes
  ShaderEffect item ...................... yes
  Sprite item ............................ yes
Qt Scxml:
  ECMAScript data model for QtScxml ...... yes
Qt Gamepad:
  SDL2 ................................... no
Qt 3D GeometryLoaders:
  Autodesk FBX ........................... no
Qt Wayland Client ........................ no
Qt Wayland Compositor .................... no
Qt Bluetooth:
  BlueZ .................................. no
  BlueZ Low Energy ....................... no
  Linux Crypto API ....................... no
  WinRT Bluetooth API (desktop & UWP) .... no
Qt Sensors:
  sensorfw ............................... no
Qt Quick Controls 2:
  Styles ................................. Default Fusion Imagine Material Universal
Qt Quick Templates 2:
  Hover support .......................... yes
  Multi-touch support .................... yes
Qt Positioning:
  Gypsy GPS Daemon ....................... no
  WinRT Geolocation API .................. no
Qt Location:
  Qt.labs.location experimental QML plugin . yes
  Geoservice plugins:
    OpenStreetMap ........................ yes
    HERE ................................. yes
    Esri ................................. yes
    Mapbox ............................... yes
    MapboxGL ............................. no
    Itemsoverlay ......................... yes
QtXmlPatterns:
  XML schema support ..................... yes
Qt Multimedia:
  ALSA ................................... no
  GStreamer 1.0 .......................... no
  GStreamer 0.10 ......................... no
  Video for Linux ........................ yes
  OpenAL ................................. no
  PulseAudio ............................. no
  Resource Policy (libresourceqt5) ....... no
  Windows Audio Services ................. no
  DirectShow ............................. no
  Windows Media Foundation ............... no
Qt Tools:
  QDoc ................................... no
Qt WebEngine:
  Embedded build ......................... yes
  Full debug information ................. no
  Pepper Plugins ......................... no
  Printing and PDF ....................... no
  Proprietary Codecs ..................... no
  Spellchecker ........................... yes
  Native Spellchecker .................... no
  WebRTC ................................. no
  Use System Ninja ....................... no
  Geolocation ............................ yes
  WebChannel support ..................... yes
  Use v8 snapshot ........................ yes
  Kerberos Authentication ................ no
  Support qpa-xcb ........................ no
  Building v8 snapshot supported ......... no
  Use ALSA ............................... no
  Use PulseAudio ......................... no
  Optional system libraries used:
    re2 .................................. no
    icu .................................. no
    libwebp, libwebpmux and libwebpdemux . no
    opus ................................. no
    ffmpeg ............................... no
    libvpx ............................... no
    snappy ............................... no
    glib ................................. no
    zlib ................................. no
    minizip .............................. no
    libevent ............................. no
    jsoncpp .............................. no
    protobuf ............................. no
    libxml2 and libxslt .................. no
    lcms2 ................................ no
    png .................................. no
    JPEG ................................. no
    harfbuzz ............................. no
    freetype ............................. no
  Required system libraries:
    fontconfig ........................... no
    dbus ................................. no
    nss .................................. no
    khr .................................. no
    glibc ................................ yes
  Required system libraries for qpa-xcb:
    x11 .................................. no
    libdrm ............................... no
    xcomposite ........................... no
    xcursor .............................. no
    xi ................................... no
    xtst ................................. no

Note: Also available for Linux: linux-clang linux-icc

Note: No wayland-egl support detected. Cross-toolkit compatibility disabled.

Note: The following modules are not being compiled in this configuration:
    3dcore
    3drender

WARNING: QDoc will not be compiled, probably because libclang could not be located. This means that you cannot build the Qt documentation.

Either ensure that llvm-config is in your PATH environment variable, or set LLVM_INSTALL_DIR to the location of your llvm installation.
On Linux systems, you may be able to install libclang by installing the libclang-dev or libclang-devel package, depending on your distribution.
On macOS, you can use Homebrew's llvm package.
On Windows, you must set LLVM_INSTALL_DIR to the installation path.

WARNING: gperf is required to build QtWebEngine.

WARNING: bison is required to build QtWebEngine.

WARNING: flex is required to build QtWebEngine.

WARNING: Thumb instruction set is required to build ffmpeg for QtWebEngine.

Qt is now configured for building. Just run 'make'.
Once everything is built, you must run 'make install'.
Qt will be installed into '/opt/qt5.12.9_hi3516dv300'.

Prior to reconfiguration, make sure you remove any leftovers from
the previous build.
```
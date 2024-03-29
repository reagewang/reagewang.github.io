qt487源码编译configure embedded帮助信息

../qt-everywhere-opensource-src-4.8.7/configure -embedded --help
Usage:  configure [-h] [-prefix <dir>] [-prefix-install] [-bindir <dir>] [-libdir <dir>]
        [-docdir <dir>] [-headerdir <dir>] [-plugindir <dir> ] [-importdir <dir>] [-datadir <dir>]
        [-translationdir <dir>] [-sysconfdir <dir>] [-examplesdir <dir>]
        [-demosdir <dir>] [-buildkey <key>] [-release] [-debug]
        [-debug-and-release] [-developer-build] [-shared] [-static] [-no-fast] [-fast] [-no-largefile]
        [-largefile] [-no-exceptions] [-exceptions] [-no-accessibility]
        [-accessibility] [-no-stl] [-stl] [-no-sql-<driver>] [-sql-<driver>]
        [-plugin-sql-<driver>] [-system-sqlite] [-no-qt3support] [-qt3support]
        [-platform] [-D <string>] [-I <string>] [-L <string>] [-help]
        [-qt-zlib] [-system-zlib] [-no-gif] [-no-libtiff] [-qt-libtiff] [-system-libtiff]
        [-no-libpng] [-qt-libpng] [-system-libpng] [-no-libmng] [-qt-libmng]
        [-system-libmng] [-no-libjpeg] [-qt-libjpeg] [-system-libjpeg] [-make <part>]
        [-nomake <part>] [-R <string>]  [-l <string>] [-no-rpath]  [-rpath] [-continue]
        [-verbose] [-v] [-silent] [-no-nis] [-nis] [-no-cups] [-cups] [-no-iconv]
        [-iconv] [-no-pch] [-pch] [-no-dbus] [-dbus] [-dbus-linked] [-no-gui]
        [-no-separate-debug-info] [-no-mmx] [-no-3dnow] [-no-sse] [-no-sse2]
        [-no-sse3] [-no-ssse3] [-no-sse4.1] [-no-sse4.2] [-no-avx] [-no-neon]
        [-qtnamespace <namespace>] [-qtlibinfix <infix>] [-separate-debug-info] [-armfpa]
        [-no-optimized-qmake] [-optimized-qmake] [-no-xmlpatterns] [-xmlpatterns]
        [-no-multimedia] [-multimedia] [-no-phonon] [-phonon] [-no-phonon-backend] [-phonon-backend]
        [-no-media-backend] [-media-backend] [-no-audio-backend] [-audio-backend] 
        [-no-openssl] [-openssl] [-openssl-linked]
        [-no-gtkstyle] [-gtkstyle] [-no-svg] [-svg] [-no-webkit] [-webkit] [-webkit-debug]
        [-no-javascript-jit] [-javascript-jit]
        [-no-script] [-script] [-no-scripttools] [-scripttools] 
        [-no-declarative] [-declarative] [-no-declarative-debug] [-declarative-debug]
        [additional platform specific options (see below)]


Installation options:

    -qpa [name] ......... This will enable the QPA build.
                          QPA is a window system agnostic implementation of Qt.
                          If [name] is given, sets the default QPA platform (e.g xcb, cocoa).

 These are optional, but you may specify install directories.

    -prefix <dir> ...... This will install everything relative to <dir>
                         (default /usr/local/Trolltech/QtEmbedded-4.8.7)

    -hostprefix [dir] .. Tools and libraries needed when developing
                         applications are installed in [dir]. If [dir] is
                         not given, the current build directory will be used.

  * -prefix-install .... Force a sandboxed "local" installation of
                         Qt. This will install into
                         /usr/local/Trolltech/QtEmbedded-4.8.7, if this option is
                         disabled then some platforms will attempt a
                         "system" install by placing default values
                         in a system location other than PREFIX.

 You may use these to separate different parts of the install:

    -bindir <dir> ......... Executables will be installed to <dir>
                            (default PREFIX/bin)
    -libdir <dir> ......... Libraries will be installed to <dir>
                            (default PREFIX/lib)
    -docdir <dir> ......... Documentation will be installed to <dir>
                            (default PREFIX/doc)
    -headerdir <dir> ...... Headers will be installed to <dir>
                            (default PREFIX/include)
    -plugindir <dir> ...... Plugins will be installed to <dir>
                            (default PREFIX/plugins)
    -importdir <dir> ...... Imports for QML will be installed to <dir>
                            (default PREFIX/imports)
    -datadir <dir> ........ Data used by Qt programs will be installed to <dir>
                            (default PREFIX)
    -translationdir <dir> . Translations of Qt programs will be installed to <dir>
                            (default PREFIX/translations)
    -sysconfdir <dir> ..... Settings used by Qt programs will be looked for in <dir>
                            (default PREFIX/etc/settings)
    -examplesdir <dir> .... Examples will be installed to <dir>
                            (default PREFIX/examples)
    -demosdir <dir> ....... Demos will be installed to <dir>
                            (default PREFIX/demos)

 You may use these options to turn on strict plugin loading.

    -buildkey <key> .... Build the Qt library and plugins using the specified
                         <key>.  When the library loads plugins, it will only
                         load those that have a matching key.

Configure options:

 The defaults (*) are usually acceptable. A plus (+) denotes a default value
 that needs to be evaluated. If the evaluation succeeds, the feature is
 included. Here is a short explanation of each option:

 *  -release ........... Compile and link Qt with debugging turned off.
    -debug ............. Compile and link Qt with debugging turned on.
    -debug-and-release . Compile and link two versions of Qt, with and without
                         debugging turned on (Mac only).

    -developer-build ... Compile and link Qt with Qt developer options (including auto-tests exporting)

    -opensource ........ Compile and link the Open-Source Edition of Qt.
    -commercial ........ Compile and link the Commercial Edition of Qt.


 *  -shared ............ Create and use shared Qt libraries.
    -static ............ Create and use static Qt libraries.

 *  -no-fast ........... Configure Qt normally by generating Makefiles for all
                         project files.
    -fast .............. Configure Qt quickly by generating Makefiles only for
                         library and subdirectory targets.  All other Makefiles
                         are created as wrappers, which will in turn run qmake.

    -no-largefile ...... Disables large file support.
 +  -largefile ......... Enables Qt to access files larger than 4 GB.

 *  -no-system-proxies . Do not use system network proxies by default.
    -system-proxies .... Use system network proxies by default.

 *  -no-exceptions ..... Disable exceptions on compilers that support it.
    -exceptions ........ Enable exceptions on compilers that support it.

    -no-accessibility .. Do not compile Accessibility support.
 *  -accessibility ..... Compile Accessibility support.

    -no-stl ............ Do not compile STL support.
 *  -stl ............... Compile STL support.

    -no-sql-<driver> ... Disable SQL <driver> entirely.
    -qt-sql-<driver> ... Enable a SQL <driver> in the QtSql library, by default
                         none are turned on.
    -plugin-sql-<driver> Enable SQL <driver> as a plugin to be linked to
                         at run time.

                         Possible values for <driver>:
                         [  db2 ibase mysql oci odbc psql sqlite sqlite2 sqlite_symbian symsql tds ]

    -system-sqlite ..... Use sqlite from the operating system.

    -no-qt3support ..... Disables the Qt 3 support functionality.
 *  -qt3support ........ Enables the Qt 3 support functionality.

    -no-xmlpatterns .... Do not build the QtXmlPatterns module.
 +  -xmlpatterns ....... Build the QtXmlPatterns module.
                         QtXmlPatterns is built if a decent C++ compiler
                         is used and exceptions are enabled.

    -no-multimedia ..... Do not build the QtMultimedia module.
 +  -multimedia ........ Build the QtMultimedia module.

    -no-audio-backend .. Do not build the platform audio backend into QtMultimedia.
 +  -audio-backend ..... Build the platform audio backend into QtMultimedia if available.

    -no-phonon ......... Do not build the Phonon module.
 +  -phonon ............ Build the Phonon module.
                         Phonon is built if a decent C++ compiler is used.
    -no-phonon-backend.. Do not build the platform phonon plugin.
 +  -phonon-backend..... Build the platform phonon plugin.

    -no-svg ............ Do not build the SVG module.
 +  -svg ............... Build the SVG module.

    -no-webkit ......... Do not build the WebKit module.
 +  -webkit ............ Build the WebKit module.
                         WebKit is built if a decent C++ compiler is used.
    -webkit-debug ...... Build the WebKit module with debug symbols.

    -no-javascript-jit . Do not build the JavaScriptCore JIT compiler.
 +  -javascript-jit .... Build the JavaScriptCore JIT compiler.

    -no-script ......... Do not build the QtScript module.
 +  -script ............ Build the QtScript module.

    -no-scripttools .... Do not build the QtScriptTools module.
 +  -scripttools ....... Build the QtScriptTools module.

    -no-declarative ..... Do not build the declarative module.
 +  -declarative ....... Build the declarative module.

    -no-declarative-debug ..... Do not build the declarative debugging support.
 +  -declarative-debug ....... Build the declarative debugging support.

    -platform target ... The operating system and compiler you are building
                         on (qws/linux-x86_64-g++).

                         See the README file for a list of supported
                         operating systems and compilers.

    -no-mmx ............ Do not compile with use of MMX instructions.
    -no-3dnow .......... Do not compile with use of 3DNOW instructions.
    -no-sse ............ Do not compile with use of SSE instructions.
    -no-sse2 ........... Do not compile with use of SSE2 instructions.
    -no-sse3 ........... Do not compile with use of SSE3 instructions.
    -no-ssse3 .......... Do not compile with use of SSSE3 instructions.
    -no-sse4.1.......... Do not compile with use of SSE4.1 instructions.
    -no-sse4.2.......... Do not compile with use of SSE4.2 instructions.
    -no-avx ............ Do not compile with use of AVX instructions.
    -no-neon ........... Do not compile with use of NEON instructions.

    -qtnamespace <name>  Wraps all Qt library code in 'namespace <name> {...}'.
    -qtlibinfix <infix>  Renames all libQt*.so to libQt*<infix>.so.

    -D <string> ........ Add an explicit define to the preprocessor.
    -I <string> ........ Add an explicit include path.
    -L <string> ........ Add an explicit library path.

    -help, -h .......... Display this information.

Third Party Libraries:

    -qt-zlib ........... Use the zlib bundled with Qt.
 +  -system-zlib ....... Use zlib from the operating system.
                         See http://www.gzip.org/zlib

    -no-gif ............ Do not compile GIF reading support.

    -no-libtiff ........ Do not compile TIFF support.
    -qt-libtiff ........ Use the libtiff bundled with Qt.
 +  -system-libtiff .... Use libtiff from the operating system.
                         See http://www.libtiff.org

    -no-libpng ......... Do not compile PNG support.
    -qt-libpng ......... Use the libpng bundled with Qt.
 +  -system-libpng ..... Use libpng from the operating system.
                         See http://www.libpng.org/pub/png

    -no-libmng ......... Do not compile MNG support.
    -qt-libmng ......... Use the libmng bundled with Qt.
 +  -system-libmng ..... Use libmng from the operating system.
                         See http://www.libmng.com

    -no-libjpeg ........ Do not compile JPEG support.
    -qt-libjpeg ........ Use the libjpeg bundled with Qt.
 +  -system-libjpeg .... Use libjpeg from the operating system.
                         See http://www.ijg.org

    -no-openssl ........ Do not compile support for OpenSSL.
 +  -openssl ........... Enable run-time OpenSSL support.
    -openssl-linked .... Enabled linked OpenSSL support.

    -ptmalloc .......... Override the system memory allocator with ptmalloc.
                         (Experimental.)

Additional options:

    -make <part> ....... Add part to the list of parts to be built at make time.
                         (libs tools examples demos docs translations)
    -nomake <part> ..... Exclude part from the list of parts to be built.

    -R <string> ........ Add an explicit runtime library path to the Qt
                         libraries.
    -l <string> ........ Add an explicit library.

    -no-rpath .......... Do not use the library install path as a runtime
                         library path.
 +  -rpath ............. Link Qt libraries and executables using the library
                         install path as a runtime library path. Equivalent
                         to -R install_libpath

    -continue .......... Continue as far as possible if an error occurs.

    -verbose, -v ....... Print verbose information about each step of the
                         configure process.

    -silent ............ Reduce the build output so that warnings and errors
                         can be seen more easily.

 *  -no-optimized-qmake ... Do not build qmake optimized.
    -optimized-qmake ...... Build qmake optimized.

    -no-gui ............ Don't build the Qt GUI library

    -no-nis ............ Do not compile NIS support.
 *  -nis ............... Compile NIS support.

    -no-cups ........... Do not compile CUPS support.
 *  -cups .............. Compile CUPS support.
                         Requires cups/cups.h and libcups.so.2.

    -no-iconv .......... Do not compile support for iconv(3).
 *  -iconv ............. Compile support for iconv(3).

    -no-pch ............ Do not use precompiled header support.
 *  -pch ............... Use precompiled header support.

    -no-dbus ........... Do not compile the QtDBus module.
 +  -dbus .............. Compile the QtDBus module and dynamically load libdbus-1.
    -dbus-linked ....... Compile the QtDBus module and link to libdbus-1.

    -reduce-relocations ..... Reduce relocations in the libraries through extra
                              linker optimizations (Qt/X11 and Qt for Embedded Linux only;
                              experimental; needs GNU ld >= 2.18).

 *  -no-separate-debug-info . Do not store debug information in a separate file.
    -separate-debug-info .... Strip debug information into a separate file.

Qt for Embedded Linux:

    -embedded <arch> .... This will enable the embedded build, you must have a
                          proper license for this switch to work.
                          Example values for <arch>: arm mips x86 generic

    -xplatform target ... The target platform when cross-compiling.

    -device-option <key=value> ... Add device specific options for the device mkspec
                                   (experimental)

    -no-feature-<feature> Do not compile in <feature>.
    -feature-<feature> .. Compile in <feature>. The available features
                          are described in src/corelib/global/qfeatures.txt

    -armfpa ............. Target platform uses the ARM-FPA floating point format.
    -no-armfpa .......... Target platform does not use the ARM-FPA floating point format.

                          The floating point format is usually autodetected by configure. Use this
                          to override the detected value.

    -little-endian ...... Target platform is little endian (LSB first).
    -big-endian ......... Target platform is big endian (MSB first).

    -host-little-endian . Host platform is little endian (LSB first).
    -host-big-endian .... Host platform is big endian (MSB first).

                          You only need to specify the endianness when
                          cross-compiling, otherwise the host
                          endianness will be used.

    -no-freetype ........ Do not compile in Freetype2 support.
    -qt-freetype ........ Use the libfreetype bundled with Qt.
 *  -system-freetype .... Use libfreetype from the operating system.
                          See http://www.freetype.org/

    -qconfig local ...... Use src/corelib/global/qconfig-local.h rather than the
                          default (full).

    -no-opengl .......... Do not support OpenGL.
    -opengl <api> ....... Enable OpenGL ES support
                          With no parameter, this will attempt to auto-detect OpenGL ES 1.x
                          or 2.x, or regular desktop OpenGL.
                          Use es1 or es2 for <api> to override auto-detection.

    -depths <list> ...... Comma-separated list of supported bit-per-pixel
                          depths, from: 1, 4, 8, 12, 15, 16, 18, 24, 32 and 'all'.

    -qt-decoration-<style> ....Enable a decoration <style> in the QtGui library,
                               by default all available decorations are on.
			       Possible values for <style>: [ styled windows default ]
    -plugin-decoration-<style> Enable decoration <style> as a plugin to be
                               linked to at run time.
			       Possible values for <style>: [  default styled windows ]
    -no-decoration-<style> ....Disable decoration <style> entirely.
                               Possible values for <style>: [ styled windows default ]

    -qt-gfx-<driver> ... Enable a graphics <driver> in the QtGui library.
                         Possible values for <driver>: [ linuxfb transformed qvfb vnc multiscreen directfb qnx integrityfb ]
    -plugin-gfx-<driver> Enable graphics <driver> as a plugin to be
                         linked to at run time.
                         Possible values for <driver>: [  ahi directfb eglnullws linuxfb powervr qvfb transformed vnc ]
    -no-gfx-<driver> ... Disable graphics <driver> entirely.
                         Possible values for <driver>: [ linuxfb transformed qvfb vnc multiscreen directfb qnx integrityfb ]

    -qt-kbd-<driver> ... Enable a keyboard <driver> in the QtGui library.
                         Possible values for <driver>: [ tty linuxinput qvfb qnx integrity ]

    -plugin-kbd-<driver> Enable keyboard <driver> as a plugin to be linked to
                         at runtime.
                         Possible values for <driver>: [  linuxinput ]

    -no-kbd-<driver> ... Disable keyboard <driver> entirely.
                         Possible values for <driver>: [ tty linuxinput qvfb qnx integrity ]

    -qt-mouse-<driver> ... Enable a mouse <driver> in the QtGui library.
                           Possible values for <driver>: [ pc linuxtp linuxinput tslib qvfb qnx integrity ]
    -plugin-mouse-<driver> Enable mouse <driver> as a plugin to be linked to
                           at runtime.
                           Possible values for <driver>: [  linuxtp pc tslib ]
    -no-mouse-<driver> ... Disable mouse <driver> entirely.
                           Possible values for <driver>: [ pc linuxtp linuxinput tslib qvfb qnx integrity ]

    -iwmmxt ............ Compile using the iWMMXt instruction set
                         (available on some XScale CPUs).
    -no-glib ........... Do not compile Glib support.
 +  -glib .............. Compile Glib support.


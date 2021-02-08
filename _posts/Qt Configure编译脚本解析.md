# Top-level installation directories: 顶级安装目录
|操作符|参数|详情|默认值|
|-|-|-|-|
|-prefix|[dir]|在目标设备上的部署目录|[/usr/local/Qt-$QT_VERSION]|
|-extprefix|[dir]|在目标设备上的安装目录|[SYSROOT/PREFIX]|
|-hostprefix|[dir]|运行在主机上的构建工具的安装目录|[EXTPREFIX]|
|-external-hostbindir|[path]|为这台机器构建的Qt工具的路径.当-platform与当前系统不匹配时,构建交叉编译||

# 注意，除-sysconfdir之外的所有目录都应该位于-prefix/-hostprefix下:
|操作符|参数|详情|默认值|
|-|-|-|-|
|-bindir|[dir]|可执行文件目录|[PREFIX/bin]|
|-headerdir|[dir]|头文件目录|[PREFIX/include]|
|-libdir|[dir]|库文件目录|[PREFIX/lib]|
|-archdatadir|[dir]|Arch 依赖的数据目录|[PREFIX]|
|-plugindir|[dir]|插件目录|[ARCHDATADIR/plugins]|
|-libexecdir|[dir]|辅助程序目录|Windows 默认[ARCHDATADIR/bin] 其它默认 [ARCHDATADIR/libexec]|
|-importdir|[dir]|QML1 imports模块 目录|[ARCHDATADIR/imports]|
|-qmldir|[dir]|QML2 imports模块 目录|[ARCHDATADIR/qml]|
|-datadir|[dir]|Arch 独立数据目录|[PREFIX]|
|-docdir|[dir]|文档目录|[DATADIR/doc]|
|-translationdir|[dir]|Qt程序的翻译数据安装目录|[DATADIR/translations]|
|`-sysconfdir`|[dir]|Qt程序使用的设置目录|[PREFIX/etc/xdg]|
|-examplesdir|[dir]|示例文件安装目录|[PREFIX/examples]|
|-testsdir|[dir]|测试文件安装目录|[PREFIX/tests]|
|-hostbindir|[dir]|主机可执行文件安装目录|[HOSTPREFIX/bin]|
|-hostlibdir|[dir]|主机库文件安装目录|[HOSTPREFIX/lib]|
|-hostdatadir|[dir]|qmake使用数据安装目录|[HOSTPREFIX]|


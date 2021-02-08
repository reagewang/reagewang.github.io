# Qt 报 virtual memory exhausted: Cannot allocate memory 错的解决方法

如果是使用 qmake 作为编译工具的话，在工程 pro 的文件中添加：

CONFIG += resources_big

这需要 Qt 5.12 和或之后的版本

如果是使用 cmake 作为编译工具的话，在 CMakeLists.txt 文件中添加：

qt5_add_big_resources(SRC_FILES resources.qrc) # big resources

这需要 Qt 5.12 和 CMake 3.9 以及之后的版本
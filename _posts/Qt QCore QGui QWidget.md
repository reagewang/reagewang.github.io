# Qt QCore QGui QWidget

QCoreApplication定义在core模块中，为应用程序提供了一个非gui的事件循环；

QGuiApplication定义在gui模块中，提供了额外的gui相关的设置，比如桌面设置，风格，字体，调色板，剪切板，光标等；

QApplication定义在widgets模块中，是QWidget相关的，能设置双击间隔，按键间隔，拖拽距离和时间，滚轮滚动行数等，能获取桌面，激活的窗口，模式控件，弹跳控件等；

如果开发的应用程序无图形界面，则使用QCoreApplication；

如果有图形界面，只使用QML实现，则使用QGuiApplication；

如果需要用到QWidget，或者QML与QWidget结果开发的，则使用QApplication；
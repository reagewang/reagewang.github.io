# Qt触摸事件

## 隐藏光标指针的功能
QT_QPA_EGLFS_HIDECURSOR(for eglfs)  1
QT_QPA_FB_HIDECURSOR (for linuxfb) 1
## 环境变量中设置设备节点名
QT_QPA_EVDEV_MOUSE_PARAMETERS = /dev/input/...
QT_QPA_EVDEV_TOUCHSCREEN_PARAMETERS = /dev/input/...
## 禁用内置的输入处理程序
QT_QPA_EGLFS_DISABLE_INPUT (for eglfs)  1
QT_QPA_FB_DISABLE_INPUT  (for linuxfb) 1
## 启用tslib支持
QT_QPA_EGLFS_TSLIB (for eglfs)  1
QT_QPA_FB_TSLIB (for linuxfb) 1
## 鼠标样式
QT_QPA_EGLFS_CURSOR  (for eglfs) 
QT_QPA_FB_CURSOR  (for linuxfb) 
## 触控笔
QT_QPA_GENERIC_PLUGINS 
## 隐藏鼠标
QT_QPA_FB_HIDECURSOR 

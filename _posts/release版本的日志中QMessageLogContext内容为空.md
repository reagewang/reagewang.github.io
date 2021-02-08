# release版本的日志中QMessageLogContext内容为空

文件信息、行数等信息在Release版本默认舍弃。我们只要在.pro文件定义一个宏

DEFINES += QT_MESSAGELOGCONTEXT
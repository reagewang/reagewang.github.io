# Q_DECLARE_METATYPE与qRegisterMetaType

## 基本理解
Q_DECLARE_METATYPE

如果要使自定义类型或其他非QMetaType内置类型在QVariant中使用，必须使用该宏。

该类型必须有公有的构造、析构、复制构造函数

qRegisterMetaType 必须使用该函数的两种情况

如果非QMetaType内置类型要在 Qt 的属性系统中使用

如果非QMetaType内置类型要在 queued 信号与槽中使用

## 二者关系

二者的代码

Q_DECLARE_METATYPE 展开后是一个特化后的类 QMetaTypeId

qRegisterMetaType 将某类型注册中MetaType系统中

## 二者的联系
QMetaTypeId的类中的成员包含对qRegisterMetaType的调用

我们知道类中的成员函数并不一定会被调用(即：该宏并不确保类型被注册到MetaType)。

通过qRegisterMetaType可以确保类型被注册

两个qRegisterMetaType 的联系

无参的qRegisterMetaType函数会通过该成员调用带参数的qRegisterMetaType()


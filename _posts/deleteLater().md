# deleteLater()

首先”deleteLater()“是QObject对象的一个函数, 要想使用此方法, 必须是一个QObject对象。

”deleteLater()“依赖于Qt的event loop机制。

如果在event loop启用前被调用, 那么event loop启用后对象才会被销毁;

如果在event loop结束后被调用, 那么对象不会被销毁;

如果在没有event loop的thread使用, 那么thread结束后销毁对象。

可以多次调用此函数。

线程安全。

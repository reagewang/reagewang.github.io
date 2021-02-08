# QT 5.15前版本QTcpSocket::error,QAbstractSocket::error信号无法关联槽

在QT5.15版本之前，无法使用

connect(this,&QAbstractSocket::error, this, &SockeClient::onError);

修改为：

connect(this,QOverload<QAbstractSocket::SocketError>::of(&QAbstractSocket::error), this, &SockeClient::onError);

QT5.15版本中，QAbstractSocket Class将原来的

void error(QAbstractSocket::SocketError socketError)

修改为：

void errorOccurred(QAbstractSocket::SocketError socketError)

理论上，现在使用

connect(this,&QAbstractSocket::errorOccurred, this, &SockeClient::onError);


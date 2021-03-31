# Qt实现HTTP的Get/Post请求
先构造一个QNetworkAccessManager对象，QNetworkAccessManager对象提供了发送QNetworkRequest网络请求和接收QNetworkReply网络回复的功能。

QNetworkAccessManager还提供了缓存和Cookie管理、代理设置等功能。

QNetworkRequest提供了对本次网络请求的封装，在本示例中只是构造了一个最简单的requset，没有进行任何参数设置。QNetworkRequest提供了很多方法来对请求进行配置，比如我们可以使用QNetworkRequest::setHeader设置请求头等。

```c++
void QtGuiApplication::onBtnGetClicked() {
	QNetworkRequest request;
	QNetworkAccessManager* naManager = new QNetworkAccessManager(this);
	QMetaObject::Connection connRet = QObject::connect(naManager, SIGNAL(finished(QNetworkReply*)), this, SLOT(requestFinished(QNetworkReply*)));
	Q_ASSERT(connRet);

	request.setUrl(QUrl("https://www.baidu.com"));
	QNetworkReply* reply = naManager->get(request);
}

//请求是异步的，当请求完成之后，会调用void requestFinished(QNetworkReply* reply);槽函数：
void QtGuiApplication::requestFinished(QNetworkReply* reply) {
	// 获取http状态码
	QVariant statusCode = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute);
	if(statusCode.isValid())
		qDebug() << "status code=" << statusCode.toInt();

	QVariant reason = reply->attribute(QNetworkRequest::HttpReasonPhraseAttribute).toString();
	if(reason.isValid())
		qDebug() << "reason=" << reason.toString();

	QNetworkReply::NetworkError err = reply->error();
	if(err != QNetworkReply::NoError) {
		qDebug() << "Failed: " << reply->errorString();
	}
	else {
		// 获取返回内容
		qDebug() << reply->readAll();
	}
}

void QtGuiApplication::onBtnPushClicked() {
	QNetworkRequest request;
	QNetworkAccessManager* naManager = new QNetworkAccessManager(this);
	QMetaObject::Connection connRet = QObject::connect(naManager, SIGNAL(finished(QNetworkReply*)), this, SLOT(requestFinished(QNetworkReply*)));
	Q_ASSERT(connRet);

	request.setUrl(QUrl("https://www.baidu.com"));
	
	QString testData = "test";
	QNetworkReply* reply = naManager->post(request, testData.toUtf8());
}

void QtGuiApplication::requestFinished(QNetworkReply* reply) {
	// 获取http状态码
	QVariant statusCode = reply->attribute(QNetworkRequest::HttpStatusCodeAttribute);
	if(statusCode.isValid())
		qDebug() << "status code=" << statusCode.toInt();

	QVariant reason = reply->attribute(QNetworkRequest::HttpReasonPhraseAttribute).toString();
	if(reason.isValid())
		qDebug() << "reason=" << reason.toString();

	QNetworkReply::NetworkError err = reply->error();
	if(err != QNetworkReply::NoError) {
		qDebug() << "Failed: " << reply->errorString();
	}
	else {
		// 获取返回内容
		qDebug() << reply->readAll();
	}
}

```
# HTTP 消息结构
Request

请求行：Request 消息中的第一行，由请求方式、请求URL、HTTP协议及版本三部分组成。

请求头：其中 Content-Type 指定了客户端发送的内容格式。例如：Content-Type: application/json，指客户端发送的内容格式为 Json。

请求体：要发送的表单数据。

Response

状态行：Response 消息中的第一行，由 HTTP 协议版本号、状态码、状态消息三部分组成。状态码用来告诉 HTTP 客户端，HTTP 服务器是否产生了预期的 Response。HTTP/1.1 中定义了 5 类状态码， 状态码由三位数字组成，第一个数字定义了响应的类别：

1XX：提示信息 - 表示请求已被成功接收，继续处理。
2XX：成功 - 表示请求已被成功接收、理解、接受。
3XX：重定向 - 要完成请求必须进行更进一步的处理。
4XX：客户端错误 - 请求有语法错误或请求无法实现。
5XX：服务器端错误 - 服务器未能实现合法的请求。

响应头：其中 Content-Type 指定了服务器返回的内容格式。例如：Content-Type: application/json，指服务器返回的内容格式为 Json。

响应体：服务器返回的内容。

# Qt支持的协议
在进行网络请求之前，首先，要查看 QNetworkAccessManager 支持的协议：
```c++
QNetworkAccessManager *manager = new QNetworkAccessManager(this);
qDebug() << manager->supportedSchemes();

// URL
QString baseUrl = "http://www.csdn.net/";

// 构造请求
QNetworkRequest request;
request.setUrl(QUrl(baseUrl));

QNetworkAccessManager *manager = new QNetworkAccessManager(this);
// 发送请求
QNetworkReply *pReplay = manager->get(request);

// 开启一个局部的事件循环，等待响应结束，退出
QEventLoop eventLoop;
QObject::connect(manager, &QNetworkAccessManager::finished, &eventLoop, &QEventLoop::quit);
eventLoop.exec();

// 获取响应信息
QByteArray bytes = pReplay->readAll();
qDebug() << bytes;
```
因为请求的过程是异步的，所以此时使用 QEventLoop 开启一个局部的事件循环，等待响应结束，事件循环退出。

注意：开启事件循环的同时，程序界面将不会响应用户操作（界面被阻塞）。

# 传递 URL 参数
你也许经常想为 URL 的查询字符串（query string）传递某种数据。如果你是手工构建 URL，那么数据会以键/值对的形式置于 URL 中，跟在一个问号的后面。例如：http://www.zhihu.com/search?type=content&q=Qt。Qt 提供了 QUrlQuery 类，可以很便利地提供这些参数。

举例来说，如果你想传递 type=content 和 q=Qt 到 http://www.zhihu.com/search，可以使用如下代码：

```c++
// URL
QString baseUrl = "http://www.zhihu.com/search";
QUrl url(baseUrl);

// key-value 对
QUrlQuery query;
query.addQueryItem("type", "content");
query.addQueryItem("q", "Qt");

url.setQuery(query);
```
通过打印输出该 URL，你能看到 URL 已被正确编码：
```c++
qDebug() << url.url();
// "http://www.zhihu.com/search?type=content&q=Qt"
```
# 更加复杂的 POST 请求
```c++
// URL
QString baseUrl = "http://httpbin.org/post";
QUrl url(baseUrl);

// 表单数据
QByteArray dataArray;
dataArray.append("key1=value1&");
dataArray.append("key2=value2");

// 构造请求
QNetworkRequest request;
request.setHeader(QNetworkRequest::ContentTypeHeader, "application/x-www-form-urlencoded");
request.setUrl(url);

QNetworkAccessManager *manager = new QNetworkAccessManager(this);

// 发送请求
manager->post(request, dataArray);
```
还可以使用 json 参数直接传递，然后它就会被自动编码：
```c++
// URL
QString baseUrl = "http://httpbin.org/post";
QUrl url(baseUrl);

// Json数据
QJsonObject json;
json.insert("User", "Qter");
json.insert("Password", "123456");

QJsonDocument document;
document.setObject(json);
QByteArray dataArray = document.toJson(QJsonDocument::Compact);

// 构造请求
QNetworkRequest request;
request.setHeader(QNetworkRequest::ContentTypeHeader, "application/json");
request.setUrl(url);

QNetworkAccessManager *manager = new QNetworkAccessManager(this);

// 连接信号槽
connect(manager, SIGNAL(finished(QNetworkReply *)), this, SLOT(replyFinished(QNetworkReply *)));

// 发送请求
manager->post(request, dataArray);
```
为了不阻塞界面，我们不再使用 QEventLoop，而用 QNetworkAccessManager 对应的信号，当响应结束就会发射 finished() 信号，将其链接到对应的槽函数上即可。

# 定制请求头
如果你想为请求添加 HTTP 头部，只要简单地调用 setHeader() 就可以了。

例如，发送的请求时，使用的 User-Agent 是 Mozilla/5.0 , 为了方便以后追踪版本信息，可以将软件的版本信息写入到 User-Agent 中。
```c++
QNetworkRequest request;
request.setHeader(QNetworkRequest::UserAgentHeader, "my-app/0.0.1");
```
当然，除了 User-Agent 之外，QNetworkRequest::KnownHeaders 还包含其他请求头，它就是为 HTTP 头部而生的。根据 RFC 2616， HTTP 头部是大小写不敏感。

如果 QNetworkRequest::KnownHeaders 不满足需要，使用 setRawHeader()。

# 响应内容
要获取响应的内容，可以调用 readAll()，由于上述的 POST 请求返回的数据为 Json 格式，将响应结果先转化为 Json，然后再对其解析：
```c++
void replyFinished(QNetworkReply *reply)
{
    // 获取响应信息
    QByteArray bytes = reply->readAll();

    QJsonParseError jsonError;
    QJsonDocument doucment = QJsonDocument::fromJson(bytes, &jsonError);
    if (jsonError.error != QJsonParseError::NoError) {
        qDebug() << QStringLiteral("解析Json失败");
        return;
    }

    // 解析Json
    if (doucment.isObject()) {
        QJsonObject obj = doucment.object();
        QJsonValue value;
        if (obj.contains("data")) {
            value = obj.take("data");
            if (value.isString()) {
                QString data = value.toString();
                qDebug() << data;
            }
        }
    }
}
```
响应的内容可以是 HTML 页面，也可以是字符串、Json、XML等。最上面所发送的 GET 请求 获取的就是 CSDN 的首页 HTML。

# 响应状态码
我们可以检测响应状态码：
```c++
QVariant variant = pReplay->attribute(QNetworkRequest::HttpStatusCodeAttribute);
if (variant.isValid())
    qDebug() << variant.toInt();
// 200
```
# 响应头
进入 Response Headers：    

可以看到包含很多，和上面一样，使用任意 QNetworkRequest::KnownHeaders 来访问这些响应头字段。例如，Content Type：
```c++
QVariant variant = pReplay->header(QNetworkRequest::ContentTypeHeader);
if (variant.isValid())
    qDebug() << variant.toString();
// "text/html; charset=utf-8"
```
如果 QNetworkRequest::KnownHeaders 不满足需要，可以使用 rawHeader()。例如，响应包含了登录后的 TOKEN，位于原始消息头中：
```c++
if (reply->hasRawHeader("TOKEN"))
    QByteArray byte = reply->rawHeader("TOKEN");
```
它还有一个特殊点，那就是服务器可以多次接受同一 header，每次都使用不同的值。Qt 会将它们合并，这样它们就可以用一个映射来表示出来

接收者可以合并多个相同名称的 header 栏位，把它们合为一个 "field-name: field-value" 配对，将每个后续的栏位值依次追加到合并的栏位值中，用逗号隔开即可，这样做不会改变信息的语义。

# 错误
如果请求的处理过程中遇到错误（如：DNS 查询失败、拒绝连接等）时，则会产生一个 QNetworkReply::NetworkError。

例如，我们将 URL 改为这样：
```c++
QString baseUrl = "http://www.csdnQter.net/";
```
发送请求后，由于主机名无效，必然会发生错误，根据 error() 来判断：
```c++
QNetworkReply::NetworkError error = pReplay->error();
switch (error) {
case QNetworkReply::ConnectionRefusedError:
    qDebug() << QStringLiteral("远程服务器拒绝连接");
    break;
case QNetworkReply::HostNotFoundError:
    qDebug() << QStringLiteral("远程主机名未找到（无效主机名）");
    break;
case QNetworkReply::TooManyRedirectsError:
    qDebug() << QStringLiteral("请求超过了设定的最大重定向次数");
    break;
deafult:
    qDebug() << QStringLiteral("未知错误");
}
// "远程主机名未找到（无效主机名）"
```
这时，会产生一个 QNetworkReply::HostNotFoundError 错误。

注意： QNetworkReply::TooManyRedirectsError 是 5.6 中引进的，默认限制是 50，可以使用 QNetworkRequest::setMaxRedirectsAllowed() 来改变。

如果要获取到可读的错误信息，使用：
```c++
QString strError = pReplay->errorString();
qDebug() << strError;
// "Host www.csdnqter.net not found"
```

# 关于QT网络库发送https请求遇到TLS initialization failed错误的解决方法
使用QT网络库向google发送https请求结果失败，错误信息为：”TLS initialization failed“，其实是缺少openssl的支持，网上也提供了几种解决方法但并不是对每个人都适用(Qt版本差异)
## 方法一：
将（QT安装目录）目录：C:\Qt\Qt5.12.9\Tools\mingw730_64\opt\bin下的ssleay32.dll和libeay32.dll拷贝到exe根目录，但是这个方法在我这个QT版本行不通，Qt5.6.0以下版本应该没问题，因为跟QT版本有很大关系，我的项目需要SSL版本是1.1.1d(实际用的1.1.1g也可以)

这里提供查看支持SSL版本方法
```c++
qDebug()<<QSslSocket::supportsSsl();//是否支持ssl
qDebug()<<QSslSocket::sslLibraryBuildVersionString();//依赖的ssl版本
```
另外openssl在1.0.x之前的版本中,文件为libeay32.dll和ssleay32.dll,在1.1.x之后的版本中，名字是libssl.dll和libcrypto.dll

## 方法二
进入这个网站：https://slproweb.com/products/Win32OpenSSL.html 下载OpenSSL安装
我需要的是1.1.1d版本，但这里只有1.1.1g，因此我下载这个的exe进行安装。安装后在QT项目的.pro文件中加入：
```c++
INCLUDEPATH += C:/OpenSSL-Win32/include
```
这样就可以了，可以测试一下。
打包的时候需要将（1.0以下版本是ssleay32.dll和libeay32.dll）（1.1版本对应的是libcrypto和libssl 的dll文件）复制到应用目录

# QNetworkRequest 请求http协议地址时返回 301 Moved Permanently 错误

通过reply->rawHeader("Location");获取知道需要调转https地址

修改QNetworkRequest url地址为https协议返回 TLS initialization failed 错误

qt.network.ssl: QSslSocket::connectToHostEncrypted: TLS initialization failed

1、对于Qt mingw版本用户来说一般在：

C:\Qt\Qt 5.xx.x\Tools\mingw730_32\opt\bin（32位）

或者：

C:\Qt\Qt 5.xx.x\Tools\mingw730_64\opt\bin（64位）

目录（以实际安装路径为准）可以找到  libeay32.dll ssleay32.dll 两个文件

1）复制这两个文件到Qt工程生成可执行文件的目录（即exe文件同目录下）中去，再次运行，错误消失

2）复制这两个文件到C:\Qt\Qt 5.xx.x\Qt 5.xx.x\mingwxxx_xx\bin 的目录下，再次运行，错误消失

上方式一般对于mingw版本来说基本可以解决

# QNetworkReply报SSL handshake failed错误
```c++
QSslConfiguration conf = request.sslConfiguration();
conf.setPeerVerifyMode(QSslSocket::VerifyNone);
request.setSslConfiguration(conf);
```
# QT生成与解析JSON数据，包含JSON数组

```c++
// 构建 Json 数组 - Version
QJsonArray versionArray;
versionArray.append(4.8);
versionArray.append(5.2);
versionArray.append(5.7);
 
// 构建 Json 对象 - Page
QJsonObject pageObject;
pageObject.insert("Home", "https://www.qt.io/");
pageObject.insert("Download", "https://www.qt.io/download/");
pageObject.insert("Developers", "https://www.qt.io/developers/");
 
// 构建 Json 对象
QJsonObject json;
json.insert("Name", "Qt");
json.insert("Company", "Digia");
json.insert("From", 1991);
json.insert("Version", QJsonValue(versionArray));
json.insert("Page", QJsonValue(pageObject));
 
// 构建 Json 文档
QJsonDocument document;
document.setObject(json);
QByteArray byteArray = document.toJson(QJsonDocument::Compact);
QString strJson(byteArray);
 
qDebug() << strJson;
```
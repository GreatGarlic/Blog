---
title: Qt 的 Json 读取工具 JsonReader
date: 2016-11-10 19:42:17
tags: Qt
---

读取下面 Json 文件 x.json 中 admin 属性下的 roles 属性，使用工具类 JsonReader，支持带 "." 的路径格式，代码如下

```cpp
JsonReader reader("x.json", true);
qDebug() << reader.getStringList("admin.roles");
```

如果使用原生的 QJsonDocument 来读的话，则代码如下

```cpp
QByteArray json; // json 的内容
QFile file("x.json"); // Json 的文件

// [1] 读取 Json 文件内容
if (file.open(QIODevice::ReadOnly | QIODevice::Text)) {
    json = file.readAll();
} else {
    return -1;
}

// [2] 解析 Json 得到根节点的 QJsonObject
QJsonParseError error;
QJsonDocument jsonDocument = QJsonDocument::fromJson(json, &error);
QJsonObject root = jsonDocument.object();

if (QJsonParseError::NoError != error.error) {
    qDebug() << error.errorString() << ", Offset: " << error.offset;
}

// [3] 按路径访问得到 roles 的数组
QJsonArray roles = root.value("admin").toObject().value("roles").toArray();

QStringList result;

// [4] 遍历 roles 数组得到数组中的所有字符串
for (QJsonArray::const_iterator iter = roles.begin(); iter != roles.end(); ++iter) {
    QJsonValue value = *iter;
    result << value.toString();
}

qDebug() << result;
```

相比之下，JsonReader 简单很多，省去了很多繁杂的步骤。

<!--more-->

## 创建 JsonReader 对象
JsonReader 支持从文件中读取 Json，也支持字符串格式的 Json

* 使用 Json 文件创建 JsonReader

    ```cpp
    JsonReader reader("x.json", true);
    ```
* 使用 Json 字符串创建 JsonReader

    ```cpp
    JsonReader reader2("{\"data\": {\"userId\": 12345}}");
    ```

---

## x.json

```json
{
    "status": 0,
    "message": "Congratulation",
    "admin": {
        "username": "Alice",
        "password": "Secret",
        "roles": ["ADMIN", "USER", "SUPERADMIN"]
    },
    "roomEnrollmentList": [{
        "enrollmentId": 5094,
        "examUId": "10000091",
        "examineeName": "马超",
        "subjectName": "普通逻辑"
    }, {
        "enrollmentId": 5103,
        "examUId": "10000100",
        "examineeName": "肖俊彦",
        "subjectName": "数值计算"
    }]
}
```

## main.cpp
```cpp
#include <QApplication>
#include <QDebug>
#include <QJsonArray>
#include <QDebug>
#include "JsonReader.h"

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);

    // 从文件里读取 Json
    JsonReader reader("/Users/Biao/Documents/workspace/Qt/JsonReader/x.json", true);

    // 第一级
    qDebug() << reader.getString("message");
    qDebug() << reader.getString("non-exit", "不存在");

    // 第二级
    qDebug() << reader.getString("admin.username");
    qDebug() << reader.getString("admin.password");

    // 第二级的数组
    qDebug() << reader.getStringList("admin.roles");

    // 第二级的数组中的值
    QJsonArray array = reader.getJsonArray("roomEnrollmentList");
    QJsonObject fromNode = array.at(0).toObject();
    qDebug() << reader.getString("examineeName", "", fromNode); // 传入开始查找的节点

    // 从字符串里读取 Json
    JsonReader reader2("{\"data\": {\"userId\": 12345}}");
    qDebug() << reader2.getInt("data.userId");

    return a.exec();
}
```

输出:

```
"Congratulation"
"不存在"
"Alice"
"Secret"
("ADMIN", "USER", "SUPERADMIN")
"马超"
12345
```

## JsonReader.h
```cpp
#ifndef JSONREADER_H
#define JSONREADER_H

#include <QJsonObject>

/**
 * Qt 的 Json 读取函数访问多层次的属性不够方便，这个类的目的就是能够使用带 "." 的路径格式访问 Json 的多级属性，例如
 * "id" 访问的是根节点下的 id，"user.address.street" 访问根节点下 user 的 address 的 street 的属性值。
 *
 * Json 例子：
 * {
 *     "id": 18191,
 *     "user": {
 *         "address": {
 *             "street": "Wiessenstrasse",
 *             "postCode": 100001
 *         },
 *         "childrenNames": ["Alice", "Bob", "John"]
 *     }
 * }
 *
 * 访问 id:     JsonReader.getInt("id")，返回 18191。
 * 访问 street: JsonReader.getString("user.address.street")，返回 "Wiessenstrasse"。
 * 访问 childrenNames: JsonReader.getStringList("user.childrenNames") 得到字符串列表("Alice", "Bob", "John")。
 *
 * 如果要找的属性不存在，则返回指定的默认值，如 "database.username.firstName" 表示的属性不存在，
 * 调用 JsonReader.getString("database.username.firstName", "defaultName")，由于要访问的属性不存在，
 * 得到的是一个空的 QJsonValue，所以返回我们指定的默认值 "defaultName"。
 *
 * 注意: Json 文件要使用 UTF-8 编码。
 */
class JsonReader {
public:
    /**
     * 使用 Json 字符串或者从文件读取 Json 内容创建 JsonReader 对象。
     * 如果 isFile 为 true，则 jsonOrJsonFilePath 为文件的路径
     * 如果 isFile 为 false，则 jsonOrJsonFilePath 为 Json 的字符串内容
     *
     * @param jsonOrJsonFilePath Json 的字符串内容或者 Json 文件的路径
     * @param isFile 为 true，则 jsonOrJsonFilePath 为文件的路径，为 false 则 jsonOrJsonFilePath 为 Json 的字符串内容
     */
    explicit JsonReader(const QString &jsonOrJsonFilePath, bool isFile = false);

    /**
     * 读取路径 path 对应属性的整数值
     * @param path 带 "." 的路径格
     * @param def 如果要找的属性不存在时返回的默认值
     * @param fromNode 从此节点开始查找，如果为默认值，则从 Json 的根节点开始查找
     * @return 整数值
     */
    int    getInt(const QString &path, int def = 0, const QJsonObject &fromNode = QJsonObject()) const;
    bool   getBool(const QString &path, bool def = false, const QJsonObject &fromNode = QJsonObject()) const;
    double getDouble(const QString &path, double def = 0.0, const QJsonObject &fromNode = QJsonObject()) const;

    QString     getString(const QString &path, const QString &def = QString(), const QJsonObject &fromNode = QJsonObject()) const;
    QStringList getStringList(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;
    QJsonArray  getJsonArray(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;
    QJsonValue  getJsonValue(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;
    QJsonObject getJsonObject(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;

private:
    QJsonObject root; // Json 的根节点
};

#endif // JSONREADER_H
```

## JsonReader.cpp
```cpp
#include "JsonReader.h"
#include <QFile>
#include <QDebug>
#include <QJsonArray>
#include <QJsonValue>
#include <QJsonDocument>
#include <QRegularExpression>
#include <QJsonParseError>

JsonReader::JsonReader(const QString &jsonOrJsonFilePath, bool isFile) {
    QByteArray json; // json 的内容

    // 如果传人的是 Json 文件的路径，则读取内容
    if (isFile) {
        QFile file(jsonOrJsonFilePath);

        if (file.open(QIODevice::ReadOnly | QIODevice::Text)) {
            json = file.readAll();
        } else {
            qDebug() << QString("Cannot open the file: %1").arg(jsonOrJsonFilePath);
        }
    } else {
        json = jsonOrJsonFilePath.toUtf8();
    }

    QJsonParseError error;
    QJsonDocument jsonDocument = QJsonDocument::fromJson(json, &error);
    root = jsonDocument.object();

    if (QJsonParseError::NoError != error.error) {
        qDebug() << error.errorString() << ", Offset: " << error.offset;
    }
}

int JsonReader::getInt(const QString &path, int def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toInt(def);
}

bool JsonReader::getBool(const QString &path, bool def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toBool(def);
}

double JsonReader::getDouble(const QString &path, double def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toDouble(def);
}

QString JsonReader::getString(const QString &path, const QString &def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toString(def);
}

QStringList JsonReader::getStringList(const QString &path, const QJsonObject &fromNode) const {
    QStringList result;
    QJsonArray array = getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toArray();

    for (QJsonArray::const_iterator iter = array.begin(); iter != array.end(); ++iter) {
        QJsonValue value = *iter;
        result << value.toString();
    }

    return result;
}

QJsonArray JsonReader::getJsonArray(const QString &path, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toArray();
}

QJsonObject JsonReader::getJsonObject(const QString &path, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode.isEmpty() ? root : fromNode).toObject();
}

QJsonValue JsonReader::getJsonValue(const QString &path, const QJsonObject &fromNode) const {
    QJsonObject parent(fromNode.isEmpty() ? root : fromNode);

    QStringList tokens = path.split(QRegularExpression("\\."));
    int size = tokens.size();

    // 定位到要访问的属性的 parent，
    // 如 "user.address.street"，要访问的属性 "street" 的 parent 是 "address"
    for (int i = 0; i < size - 1; ++i) {
        if (parent.isEmpty()) {
            return QJsonValue();
        }

        parent = parent.value(tokens.at(i)).toObject();
    }

    return parent.value(tokens.last());
}
```

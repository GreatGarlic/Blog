---
title: Qt 的 Json 读写工具类 Json
date: 2016-11-10 19:42:17
tags: Qt
---

读取下面 Json 文件 x.json 中 admin 属性下的 roles 属性，使用工具类 Json，支持带 "." 的路径格式，代码如下

```cpp
Json json("x.json", true);
qDebug() << json.getStringList("admin.roles");
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

相比之下，类 Json 简单很多，省去了很多繁杂的步骤。

<!--more-->

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

## Json 的使用
类 Json 支持从文件中读取 Json，也可以使用 Json 字符串初始化，支持使用带 `.` 的路径级连的读写 Json 的属性

* 使用 Json 文件创建 Json 对象

    ```cpp
    Json json("x.json", true);
    ```
* 使用 Json 字符串创建 Json 对象

    ```cpp
    Json json("{\"data\": {\"userId\": 12345}}");
    ```
* 读取 Json 使用 Json.getInt(), Json.getString() 等
* 写入 Json 使用 Json.set()
* 参考 main.cpp 中 Json 的使用用例

## 文件说明
* main.cpp 和 x.json 用于测试类 Json
* 类 Json 的实现文件为 Json.h 和 Json.cpp

## main.cpp
```cpp
#include <QDebug>
#include <QJsonArray>
#include <QJsonDocument>
#include "Json.h"

int main(int argc, char *argv[]) {
    Q_UNUSED(argc)
    Q_UNUSED(argv)

    // 使用字符串创建 Json
    Json j("{\"message\": \"Welcome\"}");
    qDebug() << j.getString("message");

    // 从文件读取 Json
    Json json("/Users/Biao/Documents/workspace/Qt/Json/x.json", true);

    // 第一级
    qDebug() << json.getString("message");
    qDebug() << json.getString("non-exit", "不存在");

    // 第二级
    qDebug() << json.getString("admin.username");
    qDebug() << json.getString("admin.password");

    // 第二级的数组
    qDebug() << json.getStringList("admin.roles");

    // 第二级的数组中的值
    QJsonArray array = json.getJsonArray("roomEnrollmentList");
    QJsonObject fromNode = array.at(0).toObject();
    qDebug() << json.getString("examineeName", "", fromNode); // 传入开始查找的节点

    // 修改 Json，创建 foo.bar.avatar
    json.set("foo.bar.avatar", "Sparta");
    qDebug() << json.getJsonValue("foo.bar");

    // 修改 Json，給 foo.bar 创建 fruit，和給 foo 创建 names 数组
    QJsonObject bar = json.getJsonObject("foo.bar");
    bar.insert("fruit", "Apple");
    json.set("foo.bar", bar);
    json.set("foo.names", QStringList() << "One" << "Two" << "Three");
    qDebug() << json.toString(QJsonDocument::Compact);

    // 保存到文件
    json.save("/Users/Biao/Desktop/xr.json");

    return 0;
}
```

输出:

```
"Welcome"
"Congratulation"
"不存在"
"Alice"
"Secret"
("ADMIN", "USER", "SUPERADMIN")
"马超"
QJsonValue(object, QJsonObject({"avatar":"Sparta"}))
"{\"admin\":{\"password\":\"Secret\",\"roles\":[\"ADMIN\",\"USER\",\"SUPERADMIN\"],\"username\":\"Alice\"},\"foo\":{\"bar\":{\"avatar\":\"Sparta\",\"fruit\":\"Apple\"},\"names\":[\"One\",\"Two\",\"Three\"]},\"message\":\"Congratulation\",\"roomEnrollmentList\":[{\"enrollmentId\":5094,\"examUId\":\"10000091\",\"examineeName\":\"马超\",\"subjectName\":\"普通逻辑\"},{\"enrollmentId\":5103,\"examUId\":\"10000100\",\"examineeName\":\"肖俊彦\",\"subjectName\":\"数值计算\"}],\"status\":0}"
```

> 查看保存的 Json 文件，比 x.json 中多出了下面的内容，说明修改与保存成功
> 
```json
"foo": {
    "bar": {
        "avatar": "Sparta",
        "fruit": "Apple"
    },
    "names": [
        "One",
        "Two",
        "Three"
    ]
}
```

## Json.h
```cpp
#ifndef JSON_H
#define JSON_H

#include <QJsonObject>
#include <QJsonDocument>

struct JsonPrivate;

/**
 * Qt 的 Json API 读写多层次的属性不够方便，这个类的目的就是能够使用带 "." 的路径格式访问 Json 的属性，例如
 * "id" 访问的是根节点下的 id，"user.address.street" 访问根节点下 user 的 address 的 street 的属性。
 *
 * Json 例子：
 * {
 *     "id": 18191,
 *     "user": {
 *         "address": {
 *             "street": "Wiessenstrasse",
 *             "postCode": "100001"
 *         },
 *         "childrenNames": ["Alice", "Bob", "John"]
 *     }
 * }
 *
 * 访问 id:     Json.getInt("id")，返回 18191
 * 访问 street: Json.getString("user.address.street")，返回 "Wiessenstrasse"
 * 访问 childrenNames: Json.getStringList("user.childrenNames") 得到字符串列表("Alice", "Bob", "John")
 * 设置 "user.address.postCode" 则可以使用 Json.set("user.address.postCode", "056231")
 *
 * 如果读取的属性不存在，则返回指定的默认值，如 "database.username.firstName" 不存在，
 * 调用 Json.getString("database.username.firstName", "defaultName")，由于要访问的属性不存在，
 * 得到的是一个空的 QJsonValue，所以返回我们指定的默认值 "defaultName"。
 *
 * 如果要修改的属性不存在，则会自动的先创建属性，然后设置它的值。
 *
 * 注意: Json 文件要使用 UTF-8 编码。
 */
class Json {
public:
    /**
     * 使用 Json 字符串或者从文件读取 Json 内容创建 Json 对象。
     * 如果 fromFile 为 true，则 jsonOrJsonFilePath 为文件的路径
     * 如果 fromFile 为 false，则 jsonOrJsonFilePath 为 Json 的字符串内容
     *
     * @param jsonOrJsonFilePath Json 的字符串内容或者 Json 文件的路径
     * @param fromFile 为 true，则 jsonOrJsonFilePath 为文件的路径，为 false 则 jsonOrJsonFilePath 为 Json 的字符串内容
     */
    explicit Json(const QString &jsonOrJsonFilePath = "{}", bool fromFile = false);
    ~Json();

    /**
     * 读取路径 path 对应属性的整数值
     *
     * @param path 带 "." 的路径格
     * @param def 如果要找的属性不存在时返回的默认值
     * @param fromNode 从此节点开始查找，如果为默认值，则从 Json 的根节点开始查找
     * @return 整数值
     */
    int         getInt(const QString &path, int def = 0, const QJsonObject &fromNode = QJsonObject()) const;
    bool        getBool(const QString &path, bool def = false, const QJsonObject &fromNode = QJsonObject()) const;
    double      getDouble(const QString &path, double def = 0.0, const QJsonObject &fromNode = QJsonObject()) const;
    QString     getString(const QString &path, const QString &def = QString(), const QJsonObject &fromNode = QJsonObject()) const;
    QStringList getStringList(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;

    QJsonArray  getJsonArray(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;
    QJsonValue  getJsonValue(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;
    QJsonObject getJsonObject(const QString &path, const QJsonObject &fromNode = QJsonObject()) const;

    /**
     * @brief 设置 path 对应的 Json 属性的值
     * @param path path 带 "." 的路径格
     * @param value 可以是整数，浮点数，字符串，QJsonValue, QJsonObject 等，具体请参考 QJsonValue 的构造函数
     */
    void set(const QString &path, const QJsonValue &value);
    void set(const QString &path, const QStringList &strings);

    /**
     * @brief 把 Json 保存到文件
     *
     * @param path 文件的路径
     */
    void save(const QString &path, QJsonDocument::JsonFormat format = QJsonDocument::Indented);

    QString toString(QJsonDocument::JsonFormat format = QJsonDocument::Indented) const;

public:
    JsonPrivate *d;
};

#endif // JSON_H
```

## JsonReader.cpp
```cpp
#include "Json.h"
#include <QFile>
#include <QTextStream>
#include <QDebug>
#include <QJsonArray>
#include <QJsonValue>
#include <QJsonDocument>
#include <QRegularExpression>
#include <QJsonParseError>

/***********************************************************************************************************************
 *                                                     JsonPrivate                                                     *
 **********************************************************************************************************************/
struct JsonPrivate {
    JsonPrivate(const QString &jsonOrJsonFilePath, bool fromFile);

    void setValue(QJsonObject &parent, const QString &path, const QJsonValue &newValue);
    QJsonValue getValue(const QString &path, const QJsonObject &fromNode) const;

    QJsonObject root; // Json 的根节点
};

JsonPrivate::JsonPrivate(const QString &jsonOrJsonFilePath, bool fromFile) {
    QByteArray json("{}"); // json 的内容

    // 如果传人的是 Json 文件的路径，则读取内容
    if (fromFile) {
        QFile file(jsonOrJsonFilePath);

        if (file.open(QIODevice::ReadOnly | QIODevice::Text)) {
            json = file.readAll();
        } else {
            qDebug() << QString("Cannot open the file: %1").arg(jsonOrJsonFilePath);
        }
    } else {
        json = jsonOrJsonFilePath.toUtf8();
    }

    // 解析 Json
    QJsonParseError error;
    QJsonDocument jsonDocument = QJsonDocument::fromJson(json, &error);
    root = jsonDocument.object();

    if (QJsonParseError::NoError != error.error) {
        qDebug() << error.errorString() << ", Offset: " << error.offset;
    }
}

// 使用递归+引用设置 Json 的值，因为 toObject() 等返回的是对象的副本，对其修改不会改变原来的对象，所以需要用引用来实现
void JsonPrivate::setValue(QJsonObject &parent, const QString &path, const QJsonValue &newValue) {
    const int indexOfDot = path.indexOf('.');
    const QString property = path.left(indexOfDot); // 第一个 . 之前的内容，如果 indexOfDot 是 -1 则返回整个字符串
    const QString subPath = (indexOfDot>0) ? path.mid(indexOfDot+1) : QString(); // 第一个 . 后面的内容

    QJsonValue subValue = parent[property];

    if(subPath.isEmpty()) {
        subValue = newValue;
    } else {
        QJsonObject obj = subValue.toObject();
        setValue(obj, subPath, newValue);
        subValue = obj;
    }

    parent[property] = subValue;
}

// 读取属性的值，如果 fromNode 为空，则从跟节点开始访问
QJsonValue JsonPrivate::getValue(const QString &path, const QJsonObject &fromNode) const {
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

/***********************************************************************************************************************
 *                                                         Json                                                        *
 **********************************************************************************************************************/
Json::Json(const QString &jsonOrJsonFilePath, bool fromFile) : d(new JsonPrivate(jsonOrJsonFilePath, fromFile)) {
}

Json::~Json() {
    delete d;
}

int Json::getInt(const QString &path, int def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toInt(def);
}

bool Json::getBool(const QString &path, bool def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toBool(def);
}

double Json::getDouble(const QString &path, double def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toDouble(def);
}

QString Json::getString(const QString &path, const QString &def, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toString(def);
}

QStringList Json::getStringList(const QString &path, const QJsonObject &fromNode) const {
    QStringList result;
    QJsonArray array = getJsonValue(path, fromNode).toArray();

    for (QJsonArray::const_iterator iter = array.begin(); iter != array.end(); ++iter) {
        QJsonValue value = *iter;
        result << value.toString();
    }

    return result;
}

QJsonArray Json::getJsonArray(const QString &path, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toArray();
}

QJsonObject Json::getJsonObject(const QString &path, const QJsonObject &fromNode) const {
    return getJsonValue(path, fromNode).toObject();
}

QJsonValue Json::getJsonValue(const QString &path, const QJsonObject &fromNode) const {
    return d->getValue(path, fromNode);
}


void Json::set(const QString &path, const QJsonValue &value) {
    d->setValue(d->root, path, value);
}

void Json::set(const QString &path, const QStringList &strings) {
    QJsonArray array;

    foreach (const QString &str, strings) {
        array.append(str);
    }

    d->setValue(d->root, path, array);
}

void Json::save(const QString &path, QJsonDocument::JsonFormat format) {
    QFile file(path);

    if (!file.open(QIODevice::WriteOnly | QIODevice::Truncate | QIODevice::Text)) {
        return;
    }

    QTextStream out(&file);
    out << QJsonDocument(d->root).toJson(format);
}

QString Json::toString(QJsonDocument::JsonFormat format) const {
    return QJsonDocument(d->root).toJson(format);
}
```

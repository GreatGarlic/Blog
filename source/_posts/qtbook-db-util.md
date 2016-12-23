---
title: 数据库访问工具 DBUtl
date: 2016-12-22 14:42:15
tags: QtBook
---
数据库访问工具 DBUtil

DBUtil 用于简化数据库的访问，只要准备好配置文件，调用 DBUtil 的静态函数就能直接得到查询数据库的结果。

**本文主要内容有:**

* 数据库访问的思考
* DBUtil 实例
* DBUtil 的 API
* DBUtil 的实现
* 把 SQL 语句放到文件里
* ORMapping<!--more-->

## 1. 数据库访问的思考

以查询数据库中 id 为 1 的 user 为例，思考访问数据库存在的问题以及优化。

**常用的访问数据库为以下几步：**

1. 设置数据库驱动和连接名
2. 设置数据库所在电脑的 IP，数据库名，访问的用户名和密码
3. 和数据库建立连接（打开数据库）
4. 创建 prepare 的 query
5. 绑定参数
6. 执行 query 访问数据库表
7. 处理 query 的执行结果
8. 关闭数据库连接

```cpp
void findUser() {
    {
        QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL", "Connection_Name");
        db.setHostName("127.0.0.1");
        db.setDatabaseName("qt");
        db.setUserName("root");
        db.setPassword("root");

        if (!db.open()) {
            qDebug() << "Connect to MySql error: " << db.lastError().text();
            return;
        }

        QSqlQuery query(db);
        query.prepare("SELECT * FROM user where id=:id");
        query.bindValue(":id", 1);
        query.exec();

        while (query.next()) {
            qDebug() << query.value("username").toString();
        }
    }
    
    QSqlDatabase::removeDatabase("Connection_Name");
}
```

**思考一下:**

* 如果要查询名字为 Alice 的 user 呢？  
  写一个函数 findUserByUsername()，在里面重复几乎和上面一样的代码。
* 如果数据库，用户名密码等变了呢？  
  是不是需要修改好多地方？在大一些的程序里，用到几十上百个访问数据库的函数很正常，如果都这么修改，想死的心都有了。

**使用前面我们实现的数据库连接池，可以简化一些：**

1. 获取数据库连接
2. 创建 prepare 的 query
3. 绑定参数
4. 执行 query 访问数据库表
5. 处理 query 的执行结果
6. 释放数据库连接

```cpp
void findUser() {
    QSqlDatabase db = ConnectionPool::openConnection();
    QSqlQuery query(db);
    query.prepare("SELECT * FROM user where id=:id");
    query.bindValue(":id", 1);
    query.exec();

    while (query.next()) {
        qDebug() << query.value("username").toString();
    }
    
    ConnectionPool::closeConnection(db);
}
```

这里没有了建立数据库连接的信息，有所进步，很好地避免了前面我们提出的问题之一。

**但是，仍然需要花费很多精力在：**

* 参数绑定（`query.bindValue()`有可能要调用很多次，忘了 `:` 等）
* 然后还要执行 `query.exec()`，`query.next()`，少一个都不行
* 最后才能得到查询结果，调用 `query.value(fieldName).toXXX()`
* 释放数据库连接（如果忘了这一步，可用连接就越来越少，导致连接泄漏）

还有很多代码都是模版式的，能不能进一步优化，像下面这样访问数据库：

```cpp
void findUser() {
    QMap<QString, QVariant> params;
    params["id"] = 1;
    
    QMap<QString, QVariant> result = DBUtil::selectMap("select * from user where id=:id", params);
    qDebug() << result["username"].toString();
}
```

只要 `SQL 语句` 和 `查询的参数`，调用 `DBUtil::selectMap()` 就能得到查询的结果，再如，查询 Alice 的 id: 

```cpp
int id = DBUtil::selectInt("select id from user where username='Alice'");
```

这里，看不到 `数据库连接`，看不到 `QSqlQuery`，看不到 `bindValues()` 等等，甚至于怎么访问数据库的我们都不知道，但得到了查询结果。

SQL 语言的特点就是：用户提出 `做什么`，而不是 `怎么做`。对于我们也是一样的，访问数据库的中间过程我们都不关心，数据才是我们关心的，给定 SQL 语句，就能得到查询结果，这是最理想的。但是在代码里有太多访问数据库的细节都是不得已而为之，那并不是我们的本意。能不能实现像上面这样简单的访问数据库，把焦点放在 `SQL 语句` 和 `结果`上，而忽略数据库访问的细节？答案当然是可以，这一章节讲解怎么实现这样的功能，我们称其为 `DBUtil`。

先介绍 `DBUtil` 的使用，如果对 `DBUtil` 的实现不感兴趣，可以忽略实现部分的内容，这并不会影响 `DBUtil` 的使用，有兴趣的话可以继续阅读后面实现部分。

## 2. DBUtil 实例
user 表的数据，下面的例子里需要使用：

|  id  | username | password | email         | mobile |
| :--: | -------- | -------- | ------------- | ------ |
|  1   | Alice    | passw0rd | NULL          | NULL   |
|  2   | Bob      | Passw0rd | bob@gmail.com | NULL   |
|  3   | Josh     | Pa88w0rd | NULL          | NULL   |

```cpp
#include "db/DBUtil.h"
#include "db/ConnectionPool.h"

#include <QDebug>

void useDBUtil();

int main(int argc, char *argv[]) {
    useDBUtil();
    Singleton<ConnectionPool>::getInstance().destroy(); // 销毁连接池，释放数据库连接
    return 0;
}

void useDBUtil() {
    // 1. 查找 Alice 的 ID
    qDebug() << "\n1. 查找 Alice 的 ID";
    qDebug() << DBUtil::selectInt("select id from user where username='Alice'");
    qDebug() << DBUtil::selectVariant("select id from user where username='Alice'").toInt();

    // 2. 查找 Alice 的密码
    qDebug() << "\n2. 查找 Alice 的密码";
    qDebug() << DBUtil::selectString("select password from user where username='Alice'");
    qDebug() << DBUtil::selectMap("select password from user where username='Alice'")["password"].toString();

    // 3. 查找 Alice 的所有信息，如名字，密码，邮件等
    qDebug() << "\n3. 查找 Alice 的所有信息，如名字，密码，邮件等";
    qDebug() << DBUtil::selectMap("select * from user where username='Alice'");

    // 4. 查找 Alice 和 Bob 的所有信息，如名字，密码，邮件等
    qDebug() << "\n4. 查找 Alice 和 Bob 的所有信息，如名字，密码，邮件等";
    qDebug() << DBUtil::selectMaps("select * from user where username='Alice' or username='Bob'");

    // 5. 查找 Alice 和 Bob 的密码
    qDebug() << "\n5. 查找 Alice 和 Bob 的密码";
    qDebug() << DBUtil::selectStrings("select password from user where username='Alice' or username='Bob'");

    // 6. 查询时使用命名参数
    qDebug() << "\n6. 查询时使用命名参数";
    QMap<QString, QVariant> params;
    params["id"] = 1;

    qDebug() << DBUtil::selectMap("select * from user where id=:id", params);
    qDebug() << DBUtil::selectString("select username from user where id=:id", params);
}
```

`程序输出`：
> 1. 查找 Alice 的 ID  
>    1  
>    1  
>
> 2. 查找 Alice 的密码  
>    "passw0rd"  
>    "passw0rd"  
>
> 3. 查找 Alice 的所有信息，如名字，密码，邮件等  
>    QMap(("email", QVariant(QString, "") ) ( "id" ,  QVariant(int, 1) ) ( "mobile" ,  QVariant(QString, "") ) ( "password" ,  QVariant(QString, "passw0rd") ) ( "username" ,  QVariant(QString, "Alice") ) )  
>
> 4. 查找 Alice 和 Bob 的所有信息，如名字，密码，邮件等  
>    (QMap(("email", QVariant(QString, "") ) ( "id" ,  QVariant(int, 1) ) ( "mobile" ,  QVariant(QString, "") ) ( "password" ,  QVariant(QString, "passw0rd") ) ( "username" ,  QVariant(QString, "Alice") ) ) , QMap(("email", QVariant(QString, "bob@gmail.com") ) ( "id" ,  QVariant(int, 2) ) ( "mobile" ,  QVariant(QString, "") ) ( "password" ,  QVariant(QString, "Passw0rd") ) ( "username" ,  QVariant(QString, "Bob") ) ) )  
>
> 5. 查找 Alice 和 Bob 的密码  
>    ("passw0rd", "Passw0rd")  
>
> 6. 查询时使用命名参数  
>    QMap(("email", QVariant(QString, "") ) ( "id" ,  QVariant(int, 1) ) ( "mobile" ,  QVariant(QString, "") ) ( "password" ,  QVariant(QString, "passw0rd") ) ( "username" ,  QVariant(QString, "Alice") ) )   
>    "Alice"

## 3. DBUtil 的 API
**DBUtil 的 SQL 语句和参数的格式：**

1. `QVariantMap` 等价于 `QMap<QString, QVariant>`

    ```
    typedef QMap<QString, QVariant> QVariantMap;
    ```
2. 参数 `sql` 可以是一个简单的 SQL 语句，不带参数，也可以带命名参数，如  

    ```
    QString sql1 = "select id from user where username='Alice'";
    QString sql2 = "select * from user where id=:id";
    ```

3. 如果 `sql` 带有命名参数，才需要 `params`，key 是命名参数的名字（不要冒号），value 是参数的值，如

    ```
    QString sql = "select * from user where id=:id";
    QVariantMap params;  
    params["id"] = 1;
    ```

**DBUtil API 说明：**

| 函数名            | 说明                                       |
| -------------- | ---------------------------------------- |
| insert         | static int `insert`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 执行插入语句，并返回插入行的 id |
| update         | static bool `update`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 执行更新语句 (update 和 delete 语句都是更新语句)，返回 true 为执行成功，失败则返回 false |
| selectMap      | static QVariantMap `selectMap`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 执行查询语句，查询到一条记录，并把其映射成 map，Key 是列名，Value 是列值。如果查询结果有多条记录，返回第一条记录的 map |
| selectMaps     | static `QList<QVariantMap>` `selectMaps`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 执行查询语句，查询到多条记录的 list，并把每一条记录其映射成一个 map，Key 是列名，Value 是列值 |
| selectInt      | static int `selectInt`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个整数值，如查询记录的个数，和等. |
| selectInt64    | static qint64 `selectInt64`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个长整数值, 如果返回时间戳时很方便 |
| selectString   | static QString `selectString`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个字符串 |
| selectStrings  | static QStringList `selectStrings`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是多个字符串 |
| selectDate     | static QDate `selectDate`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个日期类型 |
| selectDateTime | static QDateTime `selectDateTime`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个日期时间类型 |
| selectVariant  | static QVariant `selectVariant`(const QString &sql, const QVariantMap &params = QVariantMap()) <br> 查询结果是一个 QVariant，可以使用它的 toXxx方法转变成对应的类型，例如 toDouble(), toBool() |

## 4. DBUtil 的配置文件
在 `config.json` 里配置创建连接池的参数，下面是 `config.json` 样例:

```json
{
    "database": {
        "debug": true,
        "type": "QMYSQL",
        "host": "127.0.0.1",
        "port": 3306,
        "database_name": "qt",
        "username": "root",
        "password": "root",
        "test_on_borrow": true,
        "test_on_borrow_sql": "SELECT 1",
        "max_wait_time": 5000,
        "max_connection_count": 5,
        "sql_files": [
            "resources/sql/user.sql",
            "resources/sql/product.sql"
        ]
    },

    "qss_files": [
        "resources/qss/button.css",
        "resources/qss/groupbox.css",
        "resources/qss/centralWidget.css"
    ]
}
```

| 变量                    | 类型      | 默认值      | 说明                                       |
| --------------------- | ------- | -------- | ---------------------------------------- |
| debug                 | bool    | false    | 为 true，输出访问数据库的调试信息，例如执行的 SQL 语句，SQL 语句用到的参数，false 则不输出 |
| type                  | string  | 无        | QSqlDatabase::addDatabase() 时指定的数据库的类型，如 QMYSQL, QSQLITE, QODBC 等 |
| host                  | string  | 无        | 数据库所在电脑的 IP                              |
| port                  | integer | 0        | 数据库的端口号，如果不设置则用默认值 0，则不设置端口号，自动使用默认端口号，例如 MySQL 的是 3306 |
| database_name         | string  | 无        | 数据库的名字，SQLite 为数据库的文件名                   |
| username              | string  | 无        | 访问数据库的用户名                                |
| password              | string  | 无        | 访问数据库的密码                                 |
| test\_on_borrow       | bool    | false    | 从数据库连接池取连接时是否测试连接有效                      |
| test\_on\_borrow_sql  | string  | SELECT 1 | 测试连接是否有效时用的 SQL 语句，如 SELECT 1            |
| max\_wait_time        | integer | 5000     | 获取连接最大等待时间，单位是毫秒，如果超时，返回一个无效的数据库连接       |
| max\_connection_count | integer | 5        | 最大创建连接数                                  |

如果有默认值，在 `config.json` 没有配置的话，会使用默认值。sql\_files 和 qss\_files 先忽略，sql\_files 下面会有涉及。

**`config.json` 放在哪里？**

1. 解压下载得到的文件 DBUtil.7z（文末有下载链接）
2. 复制 `data` 目录到可执行文件所在目录（Windows 为 .exe 所在目录，Mac 的如图所示，DBUtil 是 Mac 下的可执行文件）
3. `data/config.json` 就是我们的配置文件，程序运行的时候类 `Config` 读取配置信息，用来创建数据库连接池。

## 5. DBUtil 的实现

这里只对 DBUtil 的核心部分进行说明，其他内容请参考源码。

### 5.1. bindValues()

执行 SQL 语句前，为 prepared 的 query 绑定参数（query 的 SQL 语句里有命名参数）。

```cpp
void DBUtil::bindValues(QSqlQuery *query, const QVariantMap &params) {
    for (QVariantMap::const_iterator i=params.constBegin(); i!=params.constEnd(); ++i) {
        query->bindValue(":" + i.key(), i.value());
    }
}
```

### 5.2. getFieldNames()

执行查询 SQL 语句后，得到 query 中所有的列名。

```cpp
QStringList DBUtil::getFieldNames(const QSqlQuery &query) {
    QSqlRecord record = query.record();
    QStringList names;
    int count = record.count();

    for (int i = 0; i < count; ++i) {
        names << record.fieldName(i);
    }

    return names;
}
```

### 5.3. queryToMaps()
执行查询 SQL 语句后，得到的每一行记录映射成一个 map，key 是列名，value 是列的值，并把所有行放在 list 里。

```cpp 
QList<QVariantMap> DBUtil::queryToMaps(QSqlQuery *query) {
    QList<QVariantMap> rowMaps;
    QStringList fieldNames = getFieldNames(*query);

    while (query->next()) {
        QVariantMap rowMap;

        foreach (QString fieldName, fieldNames) {
            rowMap.insert(fieldName, query->value(fieldName));
        }

        rowMaps.append(rowMap);
    }

    return rowMaps;
}
```

### 5.4. 执行 SQL 语句，查询结果为  `QList<QVariantMap>`

```cpp
QList<QVariantMap> DBUtil::selectMaps(const QString &sql, const QVariantMap &params) {
    QSqlDatabase db = Singleton<ConnectionPool>::getInstance().openConnection(); // [1]
    QSqlQuery query(db); // [2]
    query.prepare(sql);  // [3]
    bindValues(&query, params); // [4]

    QList<QVariantMap> maps;

    if (query.exec()) { // [5]
        maps = queryToMaps(&query); // [6]
    }

    debug(query, params); // 调试使用
    Singleton<ConnectionPool>::getInstance().closeConnection(db); // [7]

    return maps;
}
```

**执行 SQL 的过程和前面描述的都差不多，包括查询、插入、更新和删除等都相似：**

1. 取得数据库连接
2. 创建 QSqlQuery 对象
3. 调用 query.prepare(sql) 表明要使用 prepared 的方式查询数据库
4. 绑定参数
5. 执行 SQL 查询
6. **处理 SQL 语句的执行结果**
7. 释放数据库连接

这 7  个步骤中，只有第 6 步处理 SQL 的查询结果的代码不一样，所以可以看到下面的 `selectVariant()` 和 `selectMaps()` 几乎一样，其它几个执行 SQL 语句相关的函数就不一一列举了。

```cpp
QVariant DBUtil::selectVariant(const QString &sql, const QVariantMap &params) {
    QSqlDatabase db = Singleton<ConnectionPool>::getInstance().openConnection();
    QSqlQuery query(db);
    query.prepare(sql);
    bindValues(&query, params);

    QVariant result;

    if (query.exec() && query.next()) {
        result = query.value(0);
    }

    debug(query, params);
    Singleton<ConnectionPool>::getInstance().closeConnection(db);

    return result;
}
```

`步骤 1, 2, 3, 4, 5, 7 都还是重复的`，虽然 DBUtil 提供访问数据库的函数不会太多，也即是说，即使上面的代码有重复的，也只是在 DBUtil 里重复，不会在其他地方发生，也还是能接受的。不过，消灭重复代码是个永远的话题，如有可能，我们将进行到底，想一下还能怎么继续简化呢？

### 5.5. 使用 Lambda 表达式进一步简化

C++11 支持 Lambda 表达式，利用 Lambda 表达式可以对上面的代码进一步简化。在类 DBUtil 里定义静态函数 `executeSql()`，它定义了访问数据库算法骨架，也就是我们前面说的访问数据库的 7 个步骤，唯一变化的是第 6 步，处理不同 SQL 语句执行的结果的逻辑不一样，所以第 6 步的实现由参数传进来的 Lambda 表达式提供（注：这就是传说中设计模式里的 `策略模式`）。

```cpp
void DBUtil::executeSql(const QString &sql, const QVariantMap &params,
        std::function<void (QSqlQuery *query)> handleResult) {
    QSqlDatabase db = Singleton<ConnectionPool>::getInstance().openConnection(); // [1]
    QSqlQuery query(db); // [2]
    query.prepare(sql);  // [3]
    bindValues(&query, params); // [4]

    if (query.exec()) { // [5]
        handleResult(&query); // [6]
    }

    debug(query, params);
    Singleton<ConnectionPool>::getInstance().closeConnection(db); // [7]
}
```

### 5.6. 最后可以简化到下面这样

```cpp
QList<QVariantMap> DBUtil::selectMaps(const QString &sql, const QVariantMap &params) {
    QList<QVariantMap> maps;
    executeSql(sql, params, [&maps](QSqlQuery *query) {
        maps = queryToMaps(query);
    });

    return maps;
}

QVariant DBUtil::selectVariant(const QString &sql, const QVariantMap &params) {
    QVariant result;
    executeSql(sql, params, [&result](QSqlQuery *query) {
        if (query->next()) {
            result = query->value(0);
        }
    });

    return result;
}

int DBUtil::insert(const QString &sql, const QVariantMap &params) {
    int id = -1;
    executeSql(sql, params, [&id](QSqlQuery *query) {
        id = query->lastInsertId().toInt(); // 插入行的主键
    });

    return id;
}

bool DBUtil::update(const QString &sql, const QVariantMap &params) {
    bool result;
    executeSql(sql, params, [&result](QSqlQuery *query) {
        result = query->lastError().type() == QSqlError::NoError;
    });

    return result;
}
```

访问数据库的函数只需要实现处理 SQL 语句执行的结果的 Lambda 表达式，其它不停重复的步骤如获取连接，释放连接，参数绑定，执行 SQL 语句等都交给 executeSql() 函数，代码更加简洁了，也减少了出错的机率。

还能再继续优化吗？也许可以，但是我的脑子已经断片！

## 6. SQL 语句放到文件里
**到目前为止，我们列举的例子中，SQL 语句都是写死在代码里的，有没有缺点？**

先看看分页语句：

```sql
1. MySQL 数据库分页
select * from 表名 limit startrow, pagesize
 
2. PostgreSQL 数据库分页
select * from 表名 limit pagesize offset startrow

3. Oracle 数据库分页
select * from (select a.*, rownum rc from 表名 where rownum<=endrow) a where a.rc>=startrow
 
4. DB2 数据库分页
select * from (
    select rownumber() over() as rc,a.* from (select * from 表名 order by 列名) 
as a) where rc between startrow and endrow
 
5. SQL Server 2000 数据库分页
select top pagesize * from 表名 
where 列名 not in (select top pagesize*page 列名 from  表名 order by 列名)
order by 列名
```

同样功能的 SQL 语句有可能在不同的数据里写法不一样，如上面的分页 SQL，如果把 SQL 语句硬编码写死在代码里，当需要把数据库从 MySQL 换到 PostgreSQL 时，就很可能需要修改代码中的 SQL 语句了，既然代码被修改了，那么就需要重新编译和发布程序。

**什么时候可能会修改 SQL 语句？**

1. 例如项目开始时决定使用 MySQL，后期更换为其它数据库的可能性很小。但是假设我们有被迫害狂想症，担心哪一天老板要求更换数据库，毕竟老板的思维可不是你我能猜透的。更换数据库后，很多 SQL 语句就得修改，例如 MySQL 的分页和 Oracle 的就不一样，尤其是高级功能相关的 SQL 语句大多都和数据库相关的，并不通用。
2. 如果 SQL 语句很长，在代码里拼出来也很不容易，分成很多行，用好多加号连起来，不但容易出错，可读性也不好。
3. 开发的时候，经常修改 SQL 语句是很正常的事。
4. 还有一个很重要的问题，SQL 语句调优，我们程序员写出来的 SQL 语句大多数时候是很低效的，毕竟我们不是专业的 DBA，不能让 DBA 来修改程序吧。

如果把 SQL 语句放在文件里而不是程序的源码里，修改 SQL 语句后，不需要修改程序源码，不需要重新编译，重启一下程序就能看到修改后的 SQL 语句执行的效果，效率非常高。由于 SQL 语句是在文件里，写很长的 SQL 语句容易得多，出错了修改也方便，不用像在源码里那样用字符串相加慢慢的拼出来。SQL 语句还可以集中管理，否则去源码里到处找出 SQL 语句来修改，漏掉一些是经常发生的事，此外还能让 DBA 帮助优化 SQL 语句。

我们提供了工具类 `Sqls` 用来读取 xml 文件里的 SQL 语句，读取 xml 文件不是这里的重点，所以就不作介绍了，把焦点放在定义 SQL 语句的 xml 文件的格式和使用上。

### 6.1. 定义 SQL 语句的 xml 文件的格式

用下面这个 SQL 语句的 xml 文件为例，包含了插入，删除，查询：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sqls namespace="User">
    <define id="fields">id, username, password, email, mobile</define>

    <sql id="findByUserId">
        SELECT <include defineId="fields"/> FROM user WHERE id=%1
        <!-- include 会引用前面使用 define 定义的内容，相当于：
            SELECT id, username, password, email, mobile FROM user WHERE id=%1
        -->
    </sql>

    <sql id="findAll">
        SELECT id, username, password, email, mobile FROM user
    </sql>

    <sql id="insert">
        INSERT INTO user (username, password, email, mobile)
        VALUES (:username, :password, :email, :mobile)
    </sql>

    <sql id="update">
        UPDATE user SET username=:username, password=:password,
            email=:email, mobile=:mobile
        WHERE id=:id
    </sql>
</sqls>
```

| 名字        |  类型  |  可选  | 说明                                       |
| --------- | :--: | :--: | ---------------------------------------- |
| sqls      |  元素  |  必须  | xml 文档的根元素                               |
| namespace |  属性  |  必须  | sqls 的属性，主要为了减少命名冲突，更好的描述当前 SQL 的作用范围    |
| define    |  元素  |  可选  | 定义一个字符串，可以被重复引用，必须在所有的 sql 元素前定义，有一个唯一的 id |
| include   |  元素  |  可选  | 使用 defineId 引用 define 定义的内容，defineId 等于 define 的 id。include 元素被对应的 define 的内容替换 |
| sql       |  元素  |  可选  | 定义一个 SQL 语句，可使用 include 引用定义好的 define，include 元素被 define 的内容替换 |

### 6.2. SQL 语句的 xml 文件的放在哪里？

还记得 `config.json` 里的 sql_files 吗？

```
    "sql_files": [
       "resources/sql/user.sql",
       "resources/sql/product.sql"
   ]
```

 **`sql_files` 的值就是定义 SQL 语句的 xml 文件的路径：**

* 可以指定多个路径
* 路径可以是绝对路径
* 路径也可以是相对于可执行文件的相对路径

下面的目录结构可以直观的看到 SQL 的 xml 文件存储的位置

```
├── DBUtil.exe
├── data
│   └── config.json
└── resources
    └── sql
        ├── product.sql
        └── user.sql
```

  

### 6.3. 程序里怎么读取 SQL 语句？

调用 static QString `Sqls::getSql`(const QString &sqlNamespace, const QString &sqlId) 读取 SQL 语句：

| 参数           | 类型                    |
| ------------ | --------------------- |
| sqlNamespace | sqls 的属性 namespace 的值 |
| sqlId        | sql 的属性 id 的值         |

```cpp
#include "db/Sqls.h"
#include "db/DBUtil.h"
#include "db/ConnectionPool.h"

#include <QDebug>

void useSqlFromFile();

int main(int argc, char *argv[]) {
    useSqlFromFile();
    Singleton<ConnectionPool>::getInstance().destroy(); // 销毁连接池，释放数据库连接
    return 0;
}

void useSqlFromFile() {
    // 读取 namespace 为 User 下，id 为 findByUserId 的 SQL 语句
    qDebug() << Singleton<Sqls>::getInstance().getSql("User", "findByUserId");
    qDebug() << Singleton<Sqls>::getInstance().getSql("User", "findByUserId-1"); // 找不到这条 SQL 语句会有提示
    qDebug() << Singleton<Sqls>::getInstance().selectMap(
        Singleton<Sqls>::getInstance().getSql("User", "findByUserId").arg(2));
}
```

`程序输出`：
> "加载 SQL 文件 resources/sql/user.sql"  
> "加载 SQL 文件 resources/sql/product.sql"  
> "SELECT id, username, password, email, mobile FROM user WHERE id=%1"  
> "Cannot find SQL for User::findByUserId-1"  
> ""  
> QMap(("email", QVariant(QString, "bob@gmail.com") ) ( "id" ,  QVariant(int, 2) ) ( "mobile" ,  QVariant(QString, "") ) ( "password" ,  QVariant(QString, "Passw0rd") ) ( "username" ,  QVariant(QString, "Bob") ) )   

从程序的输出里可以看到，加载了 2 个定义 SQL 语句的文件 `user.sql` 和 `product.sql`，在 namespace `User` 里找到了 id 为 `findByUserId` 的 SQL 语句，找不到 id 为 `findByUserId-1` 的 SQL 语句。

## 7. ORMapping

使用前面的技术，如下访问 user 表

```cpp
    QVariantMap userMap = DBUtil::selectMap(Sqls::getSql("User", "findByUserId").arg(2));
    int id = userMap["id"].toInt();
    QString name = userMap["username"].toString();
```

但是，要在 20 个地方查询数据库访问 user 呢？不但调用上面的代码 20 次，每次使用的时候还需要注意数据类型，总是感觉不是很方便。如果熟悉 Java，可能对 `ORMapping` 会很熟悉，即把从数据库查询到的数据映射成一个类的对象，访问 user 表数据的时候，如下：

```cpp
    User user = UserDao::findByUserId(2); // 查询数据库
    int id = user.id;
    QString name = user.username;
```

用类 UserDao 从数据库查询 user 的数据（DAO 即数据访问对象 Data Access Object 的缩写），查询到的数据映射到类 User 的对象，这样做的好处是：

* 访问数据库的操作集中在 UserDao 里，而不是分散在代码的各个地方
* 访问 user 的时候不需要每次都转换数据的类型，因为对象的属性有自己的类型
* 如果新来个架构师认为 user 表中 id 作为列名太土了，要改为 user_id，那么只需要修改 SQL 语句和映射地方就可以。如果用 map 的话，所有用到 userMap["id"] 的地方都要修改为 userMap["user\_id"]，调用的地方多了就很容易漏掉
* 而且 userMap["id"] 即使不存在也不会报错，看一下 `T & QMap::​operator[](const Key & key)` 的说明：  
  If the map contains no item with key key, the function inserts a default-constructed value into the map with key key, and returns a reference to it.  
  这种有错，但是编译器又不报错的错误，调试是非常困难的，如果是变量名的话，修改不对，编译的时候就会报错。

所以使用 `ORMapping` 的方式来访问数据库还是不错的。下面就以 User 和 UserDao 访问数据库表 user 为例来介绍 ORMapping 的实现：

* `User` 类只是用于持有数据，没有业务逻辑，所以只是简单的定义几个属性，这里为了方便，把属性定义为 public 的，也可以定义为 private 的，然后给属性定义读写函数
* `UserDao` 用于查询数据库，并且提供把查询记录得到的 map 映射成对象的函数

### 7.1. 在 `DBUtl` 类里增加 2 个函数 `selectBean()` 和 `selectBeans()`
`selectBean()` 和 `selectBeans()` 的第一个参数都是一个函数指针 `T mapToBean(const QVariantMap &rowMap)`，这个函数的作用是把一个 QVariantMap 映射成为一个对象，一般都是在 Dao 类里定义，参考 UserDao 的实现。

```cpp
    /**
     * 查询结果封装成一个对象 bean.
     * @param sql
     * @param mapToBean - 把 map 映射成对象的函数.
     * @return 返回查找到的 bean, 如果没有查找到，返回 T 的默认对象，其 id 最好是 -1，这样便于有效的对象区别。
     */
    template <typename T>
    static T selectBean(T mapToBean(const QVariantMap &rowMap), 
                        const QString &sql, 
                        const QVariantMap &params = QVariantMap()) {
        // 把 map 都映射成一个 bean 对象
        return mapToBean(selectMap(sql, params));
    }

    /**
     * 执行查询语句，查询到多个结果并封装成 bean 的 list.
     * @param sql
     * @param params
     * @param mapToBean - 把 map 映射成 bean 对象函数.
     * @return 返回 bean 的 list，如果没有查找到，返回空的 list.
     */
    template<typename T>
    static QList<T> selectBeans(T mapToBean(const QVariantMap &rowMap), 
                                const QString &sql, 
                                const QVariantMap &params = QVariantMap()) {
        QList<T> beans;
        QList<QVariantMap> rows = selectMaps(sql, params);
        QListIterator<QVariantMap> iter(rows);

        // 每一个 map 都映射成一个 bean 对象
        while (iter.hasNext()) {
            beans.append(mapToBean(iter.next()));
        }

        return beans;
    }
```

### 7.2. `User.h`

```cpp
#ifndef USER_H
#define USER_H

#include <QString>

class User {
public:
    User();

    int id;
    QString username;
    QString password;
    QString email;
    QString mobile;

    QString toString() const;
};

#endif // USER_H
```

### 7.3. `User.cpp`

```cpp
#include "User.h"

User::User() {
    id = -1;
}

QString User::toString() const {
    return QString("ID: %1, Username: %2, Password: %3, Email: %4, Mobile: %5")
            .arg(id).arg(username).arg(password).arg(email).arg(mobile);
}
```

例如我们查询 id = 10000 的 User，数据库里没有这个 User，执行查询函数 `selectBean()` 返回的是一个默认的 User 对象，那么我们就需要有一个标志判断得到的 User 是没有意义的。User 表的主键 id 我们用的是自增长的类型 (auto increment)，所以 id 不会为 -1，于是把 id 初始化为 -1，则查询得到的 User 的 id 为 -1 的话，那么说明没有查询到 id 为 10000 的 User。这只是一个小技巧，不一定都要和用这里一样的做法，但是表示数据无效的标志是有必要的，应该根据实际情况来设定这个标志。

toString() 函数是为了方便打印 User 信息，例如调用 qDebug() << user.toString() 就可以打印出 User 的全部信息，而不用 qDebug() << user.username << user.password << user.email 等这么长的调用，如果这么打印的地方太多，写起来太麻烦，除此之外没有其它的意义。

### 7.4. `UserDao.h`

```cpp
#ifndef USERDAO_H
#define USERDAO_H

class User;
class QString;
class QVariant;
template <typename T> class QList;
template <typename KT, typename VT> class QMap;

class UserDao {
public:
    static User findByUserId(int id);
    static QList<User> findtAll();

    static int  insert(const User &user);
    static bool update(const User &user);

private:
    static User mapToUser(const QMap<QString, QVariant> &rowMap);
};

#endif // USERDAO_H
```

UserDao 里所有的函数都定义为静态函数，因为 UserDao 不需要成员变量保存调用状态，所以调用这些函数的时候没有必要先创建一个对象然后调用对象的函数。mapToUser() 必须是静态函数，因为 selectBean() 和 selectBeans() 的参数的一个普通函数的指针，不接受成员函数的指针。

### 7.5. `UserDao.cpp`

```cpp
#include "UserDao.h"
#include "bean/User.h"
#include "db/DBUtil.h"
#include "db/Sqls.h"

#include <QMap>
#include <QList>

const char * const SQL_NAMESPACE_USER = "User";

User UserDao::findByUserId(int id) {
    QString sql = Singleton<Sqls>::getInstance().getSql(SQL_NAMESPACE_USER, "findByUserId").arg(id);
    return DBUtil::selectBean(mapToUser, sql);
}

QList<User> UserDao::findtAll() {
    QString sql = Singleton<Sqls>::getInstance().getSql(SQL_NAMESPACE_USER, "findAll");
    return DBUtil::selectBeans(mapToUser, sql);
}

int UserDao::insert(const User& user) {
    QString sql = Singleton<Sqls>::getInstance().getSql(SQL_NAMESPACE_USER, "insert");

    QVariantMap params;
    params["username"] = user.username;
    params["password"] = user.password;
    params["email"]    = user.email;
    params["mobile"]   = user.mobile;

    return DBUtil::insert(sql, params);
}

bool UserDao::update(const User& user) {
    QString sql = Singleton<Sqls>::getInstance().getSql(SQL_NAMESPACE_USER, "update");

    QVariantMap params;
    params["id"]       = user.id;
    params["username"] = user.username;
    params["password"] = user.password;
    params["email"]    = user.email;
    params["mobile"]   = user.mobile;

    return DBUtil::update(sql, params);
}

/**
 * 把 map 映射为 User 对象
 */
User UserDao::mapToUser(const QMap<QString, QVariant> &rowMap) {
    User user;

    user.id = rowMap.value("id", -1).toInt();
    user.username = rowMap.value("username").toString();
    user.password = rowMap.value("password").toString();
    user.email    = rowMap.value("email").toString();
    user.mobile   = rowMap.value("mobile").toString();

    return user;
}
```

操作数据库的函数 findByUserId(), findAll(), insert(), update() 等都很简单，和前面讲过的内容没什么区别，SQL 语句是从文件里读取的，唯一需要注意的就是 mapToUser() 这个函数，其作用就是把一个 map 映射成一个对象，`user.id = rowMap.value("id", -1).toInt()` 有一个默认值 -1，前面我们说过 User 的 id 为 -1，表示 User 的数据是无效的，所以这个默认值是很重要的，既然很重要而且可能会被很多地方用到，那么更好的实践是把值 -1 定义为一个常量如 const int `INVALID_ID` = -1，用到的地方用常量 INVALID_ID 而不是直接用 -1，万一要修改也方便。

### 7.6. 测试 UserDao 的使用
```cpp
#include "bean/User.h"
#include "dao/UserDao.h"
#include "db/ConnectionPool.h"

void useDao();

int main(int argc, char *argv[]) {
    useDao();
    Singleton<ConnectionPool>::getInstance().destroy(); // 销毁连接池，释放数据库连接
    return 0;
}

void useDao() {
    // 使用基于 DBUtil 封装好的 DAO 查询数据库
    User user = UserDao::findByUserId(2);
    qDebug() << user.username;
    qDebug() << user.toString();

    // 更新数据库
    user.email = "bob@gmail.com";
    qDebug() << "Update: " << UserDao::update(user);

    QList<User> users = UserDao::findtAll();
    foreach (const User &u, users) {
        qDebug() << u.toString();
    }
}
```

`程序输出`：
> "加载 SQL 文件 resources/sql/user.sql"  
> "加载 SQL 文件 resources/sql/product.sql"  
> "Bob"  
> "ID: 2, Username: Bob, Password: Passw0rd, Email: bob@gmail.com, Mobile: "  
> Update:  true  
> "ID: 1, Username: Alice, Password: passw0rd, Email: , Mobile: "  
> "ID: 2, Username: Bob, Password: Passw0rd, Email: bob@gmail.com, Mobile: "  
> "ID: 3, Username: Josh, Password: Pa88w0rd, Email: , Mobile: "  

## 8. 思考
1. 上面的例子中，映射成对象的时候，表 user 对应类 User 的对象。想一下，查询到的结果能不能不只是映射到简单成员变量？如果成员变量是另一个类的对象时怎么做？
2. 前面把 map 映射成对象是通过提供一个函数来映射，能不能实现一个自动的映射呢？  
   提示可以通过 Qt 的 `Meta System` 来实现，具体就是 `Q_PROPERTY`。
3. 每次访问 User 的数据都要从数据库里查询一遍吗？  
   数据库查询的效率很多时候是很慢的，如数据库运行在其它电脑上，网络还很慢。能不能实现一个缓存功能，如果要查询的 User 的数据已经存在缓存里了就直接从缓存取，没有才去数据库里查询。
4. 配置文件我们使用了 Json 文件，可以自行实现使用 xml 格式，ini 格式等。

> `小提示`: 类 Config，Sqls，ConnectionPool 都用到了单例模式，例如  
>
```
Singleton<ConnectionPool>::getInstance().destroy()
```
> 单例的实现和使用请参考 [单例 Singleton](/qtbook-singleton) 。

## 9. 资源下载

DBUtil 相关的代码和文件可以在这里下载到: [DBUtil.7z](/download/qtbook/db/DBUtil.7z)

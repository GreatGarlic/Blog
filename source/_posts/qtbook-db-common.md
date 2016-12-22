---
title: 数据库常用操作
date: 2016-12-21 14:26:11
tags: QtBook
---
前面的章节介绍了怎么使用 Qt 连接访问数据库 SQLite 和 MySQL，在这一节里将介绍访问数据库的常用操作，主要内容有:

* QSqlDatabase
* 查询
    * 使用 Prepared Query 查询
    * SQL 注入
    * 使用 LIKE 模糊查询
    * 解决列名冲突
* 更新
* 删除
* 事务<!--more-->

## 1. 准备数据库数据

> `小提示`  
> 1. 数据库设计，推荐每个表都有一个无意义的主键，如命名为 `id`。
> 2. 尽量不使用外键，数据的逻辑关系使用上面提到的无意义的 id 来关联，这样的好处是数据迁移的时候不需要考虑外键的因素而造成很多麻烦，数据的逻辑关系由应用程序来控制，典型的应用有如很有名的电商开发框架 Hybris。
> 3. 因为没有用外键而不能级连删除，如果担心数据库里会留下一些垃圾数据，可以用定时任务在系统负载比较轻的时候删除它们，例如晚上 3 点。
> 4. 如果时间需要根据不同的时区显示，时间相关的字段最好使用 `timestamp` 而不是 `datetime`，既存储时间的 UTC 毫秒值。  
>    例如开发一个会议管理软件，日期使用 datetime 存储，同时有中国，美国，德国人参加会议，如果大家看到开会时间是 2015-03-03 10:00:00，每个人都会自然的认为是自己当地的时间，那么开会的时候就只有你自己一个人了。使用毫秒的话，显示的时候，根据使用者的时区自动显示为当地时间。
>
> **在开始讲解之前，先准备好需要用到的数据：**

1. 创建数据库 `qt`
2. 然后在此数据库里创建 3 张表 `user`, `blog`, `comment`
3. 在每一张表里插入一些数据

**创建 `user` 表**

```sql
CREATE TABLE `user` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(256) NOT NULL,
    `password` varchar(256) NOT NULL,
    `email` varchar(256) DEFAULT NULL,
    `mobile` varchar(32) DEFAULT NULL,
    PRIMARY KEY (`id`)
)

INSERT INTO `user` (`id`, `username`, `password`, `email`, `mobile`) VALUES
(1, 'Alice', 'passw0rd', NULL, NULL),
(2, 'Bob', 'Passw0rd', NULL, NULL),
(3, 'Josh', 'Pa88w0rd', NULL, NULL);
```

| id   | username | password | email | mobile |
| ---- | -------- | -------- | ----- | ------ |
| 1    | Alice    | passw0rd | NULL  | NULL   |
| 2    | Bob      | Passw0rd | NULL  | NULL   |
| 3    | Josh     | Pa88w0rd | NULL  | NULL   |

**创建 `blog` 表**

```sql
CREATE TABLE `blog` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `user_id` int(11) NOT NULL,
    `content` text NOT NULL,
    `created_time` datetime NOT NULL,
    `last_updated_time` datetime NOT NULL,
    PRIMARY KEY (`id`)
)

INSERT INTO `blog` (`id`, `user_id`, `content`, `created_time`, `last_updated_time`) VALUES
(1, 1, 'Content of blog 1.', '2015-01-01 10:10:10', '2015-01-01 10:10:10'),
(2, 2, 'Content of blog 2.', '2015-02-02 20:20:20', '2015-02-02 20:20:20');
```

| id   | user\_id | content            | created\_time       | last\_updated\_time |
| ---- | -------- | ------------------ | ------------------- | ------------------- |
| 1    | 1        | Content of blog 1. | 2015-01-01 10:10:10 | 2015-01-01 10:10:10 |
| 2    | 2        | Content of blog 2. | 2015-02-02 20:20:20 | 2015-02-02 20:20:20 |

 **创建 `comment` 表**

```sql
CREATE TABLE `comment` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `user_id` int(11) NOT NULL,
    `blog_id` int(11) NOT NULL,
    `content` text NOT NULL,
    `created_time` datetime NOT NULL,
    PRIMARY KEY (`id`)
)

INSERT INTO `comment` (`id`, `user_id`, `blog_id`, `content`, `created_time`) VALUES
(1, 1, 1, 'Very useful.', '2015-01-02 11:11:11'),
(2, 3, 2, 'Super', '2015-03-03 23:33:33');
```

| id   | user_id | blog_id | content      | created\_time       |
| ---- | ------- | ------- | ------------ | ------------------- |
| 1    | 1       | 1       | Very useful. | 2015-01-02 11:11:11 |
| 2    | 3       | 2       | Super        | 2015-03-03 23:33:33 |

## 2. QSqlDatabase

访问数据库前必须先和数据库建立连接，Qt 里用 `QSqlDatabase` 表示一个数据库的连接（有点不习惯，既然表示的是连接，有没有觉得如果叫 QSqlConnection 会更好？可惜 Qt 不是我们设计的！），每个连接都有自己的名字 connectionName，用同一个 connectionName 得取的 QSqlDatabase 对象都是表示同一个连接。有一点需要注意，如果要在多线程里访问数据库，`每个线程都要使用不同的数据库连接`，即每个线程使用的 QSqlDatabase 的 connectionName 都不一样，否则可能会遇到很多预料不到的事。

**创建 QSqlDatabase 对象用静态函数 `QSqlDatabase::addDatabase()`**

```cpp
QSqlDatabase QSqlDatabase::addDatabase(
    const QString &type, 
    const QString &connectionName=QLatin1String(defaultConnection)
)
```

* 第一个参数 type 是指定数据库的驱动，例如 `"QSQLITE"`, `"QMYSQL"`, `"QPSQL"`
* 第二个参数 connectionName 是连接的名字，可以是任意的字符串

**获取 QSqlDatabase 对象用静态函数  `QSqlDatabase::database()`**

```cpp
QSqlDatabase QSqlDatabase::database(
    const QString &connectionName=QLatin1String(defaultConnection), 
    bool open=true
)
```

* 第一个参数 connectionName 是连接的名字，调用 `QSqlDatabase::addDatabase()` 创建连接时使用的连接名
* 第二个参数 open 为 true，返回连接前打开连接

**使用默认的 connectionName 创建和获取连接：**

```cpp
void createDefaultConnection() {
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL");
    db.setHostName("127.0.0.1");
    db.setDatabaseName("qt"); // 如果是 SQLite 则为数据库文件路径
    db.setUserName("root");   // 如果是 SQLite 不需要
    db.setPassword("root");   // 如果是 SQLite 不需要

    if (!db.open()) { // 连接数据库
        qDebug() << "Connect to MySql error: " << db.lastError().text();
        return;
    }
}

QSqlDatabase getDefaultConnection() {
    return QSqlDatabase::database();
}
```

* 不提供 connectName，Qt 使用默认的 connectName `qt_sql_default_connection`
* 调用 createDefaultConnection() 创建数据库连接
* 调用 getConnectionByName() 取得数据库连接，多次调用这条语句得到的都是同一个数据库连接

**给定 connectionName 创建和获取连接：**

```cpp
void createConnectionByName(const QString &connectionName) {
    QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL", connectionName);
    db.setHostName("127.0.0.1");
    db.setDatabaseName("qt"); // 如果是 SQLite 则为数据库文件名
    db.setUserName("root");   // 如果是 SQLite 不需要
    db.setPassword("root");   // 如果是 SQLite 不需要

    if (!db.open()) { // 连接数据库
        qDebug() << "Connect to MySql error: " << db.lastError().text();
        return;
    }
}

QSqlDatabase getConnectionByName(const QString &connectionName) {
    return QSqlDatabase::database(connectionName);
}
```

* 例如 connectName 为 "MyConnection"
* 调用 createConnectionByName("MyConnection") 创建数据库连接
* 调用 getConnectionByName("MyConnection") 取得数据库连接，多次调用这条语句得到的都是同一个数据库连接，因为连接的名字都是同一个

重复使用同一个 connectionName 创建数据库连接不会创建多个同名的连接，也不会发生错误，只是会删除已经存在的连接，然后用此 connectionName 重新创建连接。这是很有用的，如程序开始运行的时候数据库连接是好用的，数据库后来崩溃了重启，已有的数据库连接就无效了导致不能访问数据库（不过调用 `QDatabase::isOpen()` 仍然返回 true），这时还是使用同一个 connectionName 再次创建数据库连接，就可以访问数据库了。

**为了防止连接断开导致不能访问数据库，于是每次使用数据库连接的时候就用同样的 connectionName 重新创建一个数据库连接**，这种做法是不可取的，虽然保证了程序的正确性，但是效率很低，因为创建数据库连接底层是 Socket 连接，也就是创建连接时也要进行三次握手等操作。

**传递 QSqlDatabase 的栈对象而不是指针或者引用会不会占用很多栈空间，会不会由于调用它的复制构造函数而生成另一个 QSqlDatabase 对象导致什么问题？**

* 首先 `sizeof(QSqlDatabase)` 输出 8，说明 QSqlDatabase 占用 8 个字节。  
  打开 QSqlDatabase 的源码也可以看到它只有 2 个指针的成员变量 char *defaultConnection 和 QSqlDatabasePrivate *d，空间不是问题
* 其次，它的复制构造函数创建的新对象和原来的对象共享 `char *defaultConnection` 和 `QSqlDatabasePrivate *d`，使用的是浅拷贝而不是深拷贝，数据也没问题

所以可以放心的在函数之间传递 QSqlDatabase 对象而不用担心出什么问题。

## 3. QSqlQuery

使用 QSqlQuery 来执行数据库操作，它有两个重要的构造函数，大多数时候也是用这两种形式来构造 QSqlQuery 对象。

```cpp
// 如果没有或者传入一个无效的 QSqlDatabase 对象，则使用默认的数据库连接
// 如果 query 不是空字符串，则执行这个 query
QSqlQuery::QSqlQuery(const QString &query=QString(), QSqlDatabase db=QSqlDatabase())
```

* `QSqlQuery query("SELECT * FROM user")`，使用默认的数据库连接执行查询操作
* `QSqlQuery query`，则会使用默认的数据库连接创建一个 QSqlQuery 对象，但是不执行任何操作。

```cpp
// 使用指定的数据库连接创建 QSqlQuery 对象，如果数据库连接无效，则使用默认的数据库连接
QSqlQuery::QSqlQuery(QSqlDatabase db)
```

## 4. 查询操作

已经有了数据库和相关数据，了解了 QSqlDatabase 和 QSqlQuery，接下来就可以举例使用 QSqlQuery 执行数据库操作，分析可能遇到的问题以及解决办法。为了简单起见，都使用默认的数据库连接，思考一下下面的例子里怎么把默认的数据库连接换成我们自己指定的 connectionName 的连接呢？

### 4.1. 输出 user 表里所有的 id, username, password
```cpp
/**
 * 输出 user 表里所有的 id, username, password
 */
void outputIdUsernamePassword() {
    QSqlQuery query("SELECT id, username, password FROM user");

    while (query.next()) {
        qDebug() << QString("Id: %1, Username: %2, Password: %3")
                    .arg(query.value("id").toInt())
                    .arg(query.value("username").toString())
                    .arg(query.value("password").toString());
    }
}
```

输出:
> "Id: 1, Username: Alice, Password: passw0rd"  
> "Id: 2, Username: Bob, Password: Passw0rd"  
> "Id: 3, Username: Josh, Password: Pa88w0rd"  

### 4.2. 输出 username 为 Alice 的 user

```cpp
1. 函数定义
/**
 * 输出 user 其 username 等于传入的参数 username
 * @param username
 */
void outputUser(const QString &username) {
    QString sql = "SELECT * FROM user WHERE username='" + username + "'";
    QSqlQuery query(sql);

    while (query.next()) {
        qDebug() << QString("Id: %1, Username: %2, Password: %3")
                    .arg(query.value("id").toInt())
                    .arg(query.value("username").toString())
                    .arg(query.value("password").toString());
    }
}

2. 函数调用
outputUser("Alice");
```

输出：
> "Id: 1, Username: Alice, Password: passw0rd"

输出结果和我们期待的一样。但是思考一下，上面的程序有没有什么问题？
如果我们这么调用函数 `outputUser("Alice' OR '1=1")`，输出如下：
> "Id: 1, Username: Alice, Password: passw0rd"  
> "Id: 2, Username: Bob, Password: Passw0rd"  
> "Id: 3, Username: Josh, Password: Pa88w0rd"  

是不是有什么不对？`outputUser("Alice' OR '1=1")` 按我们的想法应该输出 username 等于 `Alice' OR '1=1` 的记录，即查询结果是空，但却输出了 user 表里的所有记录，完全和期望的不一样，一定是有什么地方出错了，`但是看上去都没什么问题呀，怎么都找不到错误吧`！

这里我们引入一个概念叫做 `SQL 注入攻击`：
> SQL 注入攻击指的是通过构建特殊的输入作为参数传入，而这些输入大都是 SQL 语法里的一些组合，通过执行 SQL 语句进而执行攻击者所要的操作，其主要原因是程序没有细致地过滤用户输入的数据，致使非法数据侵入系统。

上面的程序在查询一个用户的时候却输出了所有用户的信息，就是一个 `SQL 注入攻击`的例子，原因是使用的 `SQL 语句是用字符串相加拼凑出来的`，以至于参数中的特殊字符 `'` 没有被转义而拼成了 `SELECT * FROM user WHERE username='Alice' OR '1=1'`，WHERE 条件里有 `OR '1=1'`，所以条件永远为真，于是输出了所有的记录，这样做是很危险的，如果因此而泄漏了公司机密信息，老板的小宇宙就要爆发了。

### 4.3. 怎么避免`SQL 注入攻击`？

使用 `Prepared Query` 可以避免`SQL 注入攻击`。

```cpp
/**
 * 输出 user 其 username 等于传入的参数 username
 * @param username
 */
void outputUserWithPreparedQuery(const QString &username) {
    QString sql = "SELECT * FROM user WHERE username=:username";
    QSqlQuery query;    // [1] 可以传入数据库连接，但是不能传入 SQL 语句
    query.prepare(sql); // [2] 声明使用 prepqred 的方式解析 SQL 语句

    query.bindValue(":username", username); // [3] 把占位符替换为传入的参数
    query.exec(); // [4] 执行数据库操作

    while (query.next()) { // [5]
        qDebug() << QString("Id: %1, Username: %2, Password: %3")
                    .arg(query.value("id").toInt())
                    .arg(query.value("username").toString())
                    .arg(query.value("password").toString());
    }
}
```

调用 `outputUserWithPreparedQuery("Alice")` 输出：
> "Id: 1, Username: Alice, Password: passw0rd"

调用 `outputUserWithPreparedQuery("Alice' OR '1=1")` 则没有输出，因为 user 表里没有 username 为 `Alice' OR '1=1` 的记录，太好了，`SQL 注入攻击`的问题很容易就解决了。

**使用 `Prepared Query` 分为以下 5 步：**

1. 创建 QSqlQuery 对象：`QSqlQuery query;`
2. 调用 `query.prepare(sql);` 声明要使用 `Prepared Query` 的方式来解析 SQL 语句
3. 调用 `bindValue()` 函数把`占位符`替换为传入的参数：`query.bindValue(":username", username);`
4. 所有的占位符都替换好后，调用 `query.exec();` 执行 SQL 语句
5. 遍历查询结果

**占位符的格式为**：`:` 后跟一个单词，如 `:username`。

**思考一下下面几个问题：**

1. `SELECT * FROM user WHERE username=:username AND password=:password` 的占位符是什么呢？
2. Prepared Query 中 SQL 里的字符串需要用 `‘’` 括起来吗？
3. 如果传入的参数是 QDateTime 类型，占位符应该怎么写？

**答案：**

1. 这条 SQL 语句里有 2 个占位符，分别为 `:username` 和 `:password`（不要想成是 ~~username AND password=:password~~）
2. 不需要。普通 SQL 里字符串需要用 `''` 括起来，但是在这里不需要，Qt 在替换参数的时候会智能的根据参数的类型判断是否需要加上 `''`，如果传入的字符串参数里有 `'`，也会智能的将其转义，所以避免了`SQL 注入攻击`。
3. 写法也是一样的，`:` 后跟一个单词，可以看看下面的这个例子

```cpp
/**
 * 输出 2015-02-01 10:10:10 后创建的 blog.
 */
void outputBlogWithPreparedQuery() {
    QDateTime dateTime = QDateTime::fromString("2015-02-01 10:10:10", "yyyy-MM-dd hh:mm:ss");

    QSqlQuery query;
    query.prepare("SELECT * FROM blog WHERE created_time>:createdTime");
    query.bindValue(":createdTime", dateTime);
    query.exec();

    while (query.next()) {
        qDebug() << QString("Id: %1, Content: %2, Created_Time: %3")
                    .arg(query.value("id").toInt())
                    .arg(query.value("content").toString())
                    .arg(query.value("created_time").toDateTime().toString("yyyy-MM-dd hh:mm:ss"));
    }
}

```

输出：
> "Id: 2, Content: Content of blog 2., Created_Time: 2015-02-02 20:20:20"

**什么时候使用 Prepared Query 呢？插入时能使用吗？删除时能使用吗？**

1. 如果传入参数构造 SQL 语句，为了避免 `SQL 注入攻击`，所以这时需要使用
2. SQL 语句比较长时，使用 Prepared Query 的方式构造的 SQL 语句比用字符串相加的方式容易一些，可读性也好很多
3. 插入和删除语句都能用

### 4.4. 使用 LIKE 模糊查询

查询名字里有 `o` 的所有用户：`SELECT * FROM user WHERE username LIKE '%o%'`，使用

```cpp
query.prepare("SELECT * FROM user WHERE username LIKE '%:match%'");
query.bindValue(":match", "o");
```

查询结果为空，因为 :match 是字符串，替换后 o 的两边会加上 `'`，所以生存的 SQL 语句为：

```
SELECT * FROM user WHERE username LIKE '%'o'%'
```
这个 SQL 语句是错的，应该像下面这样使用：

```cpp
query.prepare("SELECT * FROM user WHERE username LIKE :match");
query.bindValue(":match", "%o%");
```

也可以使用数据库的字符串连接函数（这个函数是数据库相关的）

```cpp
query.prepare("SELECT * FROM user WHERE username LIKE CONCAT('%', :match,'%')");
query.bindValue(":match", "o");
```

> `MySql`  
> SELECT * FROM user WHERE username like CONCAT('%',:match,'%')  

> `Oracle`  
> SELECT * FROM user WHERE username like CONCAT('%',:match,'%') 或   
> SELECT * FROM user WHERE username like '%'||:match||'%'  

> `SQLServer`  
> SELECT * FROM user WHERE username like '%'+:match+'%'  

> `DB2`  
> SELECT * FROM user WHERE username like CONCAT('%',:match,'%') 或  
> SELECT * FROM user WHERE username like '%'||:match||'%'  

### 4.5. 多表查询时字段名冲突

列出所有的 user，如果 user 有 blog，则列出 blog：`SELECT * FROM user LEFT JOIN blog ON user.id = blog.user_id`。在数据库客户端执行这条 SQL 语句结果如下图：  
![](/img/qtbook/db/DB-Query-1.png)

有 2 个列名都叫 id，第一个 id 是 user 表的 id，第二个 id 是 blog 表的 id。运行下面的程序：

```cpp
void outputUserAndBlog() {
    QSqlQuery query("SELECT * FROM user LEFT JOIN blog ON user.id = blog.user_id");

    while (query.next()) {
        qDebug() << query.value("id").toInt();
    }
}
```

输出：
> 1  
> 2  
> 3  

取到的是第一列的 id，即 user 表的 id，并没有输出 blog 表的 id。如果我们想取得 blog 表的 id 应该怎么做呢？显然上面的 SQL 不行，但是我们可以给`查询字段重命名`：

```cpp
void outputUserAndBlog() {
    QSqlQuery query("SELECT user.id as user_id, blog.id as blog_id, blog.content as content "
                    "FROM user "
                    "LEFT JOIN blog ON user.id = blog.user_id");

    while (query.next()) {
        qDebug() << QString("User_Id: %1, Blog_ID: %2, Blog_Content: %3")
                    .arg(query.value("user_id").toInt())
                    .arg(query.value("blog_id").toInt())
                    .arg(query.value("content").toString());
    }
}
```

输出：
> "User\_Id: 1, Blog\_ID: 1, Blog\_Content: Content of blog 1."  
> "User\_Id: 2, Blog\_ID: 2, Blog\_Content: Content of blog 2."  
> "User\_Id: 3, Blog\_ID: 0, Blog\_Content: "  

通过给查询字段重命名的方式，解决了`多表查询时字段名冲突`的问题。

## 5. 插入操作

```cpp
/**
 * 向 user 表里插入一个 user
 * @param username
 * @param password
 */
void insertUser(const QString &username, const QString &password) {
    QSqlQuery query;
    query.prepare("INSERT INTO user (username, password) VALUES (:username, :password)");
    query.bindValue(":username", username);
    query.bindValue(":password", password);
    query.exec();
}
```

调用 `insertUser("Qter", "secret")` 可以看到数据库里多出了刚才插入的数据。

| id   | username | password | email | mobile |
| ---- | -------- | -------- | ----- | ------ |
| 1    | Alice    | passw0rd | NULL  | NULL   |
| 2    | Bob      | Passw0rd | NULL  | NULL   |
| 3    | Josh     | Pa88w0rd | NULL  | NULL   |
| 4    | Qter     | secret   | NULL  | NULL   |

## 6. 删除操作
```cpp
/**
 * 删除名字等于传入的 username 的 user
 * @param username
 */
void deleteUser(const QString &username) {
    QSqlQuery query;
    query.prepare("DELETE FROM user WHERE username=:username");
    query.bindValue(":username", username);
    query.exec();
}
```

调用 `deleteUser("Qter")` 则删除了 user 表里 username 为 `Qter` 的所有记录。

| id   | username | password | email | mobile |
| ---- | -------- | -------- | ----- | ------ |
| 1    | Alice    | passw0rd | NULL  | NULL   |
| 2    | Bob      | Passw0rd | NULL  | NULL   |
| 3    | Josh     | Pa88w0rd | NULL  | NULL   |

想像一个场景，如果使用删除 user 的代码如下

```cpp
void deleteUser2(const QString &username) {
    QSqlQuery query("DELETE FROM user WHERE username='" + username + "'");
}
```

调用 `deleteUser("Qter' OR '1=1")`，结果是什么？大神，你把 user 表清空了，想一下，这种问题叫什么，怎么解决？

## 7. 事务

> 事务（Transaction）是并发控制的单位，是用户定义的一个操作序列。这些操作要么都做，要么都不做，是一个不可分割的工作单位。通过事务，能将逻辑相关的一组操作绑定在一起，以便服务器保持数据的完整性。

**Qt 里事物相关的函数：**

1. 是否支持事务：`bool QSqlDriver::hasFeature(DriverFeature feature) const`，返回 true 表示支持事务，返回 false 表示不支持事务
2. 开启事务：`bool QSqlDatabase::​transaction()`
3. 提交事务：`bool QSqlDatabase::​commit()`
4. 回滚事务：`bool QSqlDatabase::rollback()`

```php
/**
 * 查看是否支持事物
 */
void isTransactionSupported() {
    createConnectionByName("Buda");
    QSqlDatabase db = getConnectionByName("Buda");

    if (db.driver()->hasFeature(QSqlDriver::Transactions)) {
        qDebug() << "支持事务";
    } else {
        qDebug() << "不支持事务";
    }
}
```

**下面的例子展示了事务的使用：**

1. 开启事物
2. 操作数据库
3. 提交事务或者回滚事务

```cpp
void transactionDemo() {
    createConnectionByName("Buda");
    QSqlDatabase db = getConnectionByName("Buda");

    db.transaction(); // [1] 开启事务

    // [2] 执行 SQL，在事务提交前更新不会提交到数据库
    QSqlQuery query1("INSERT INTO user (id, username, password) VALUES (1, 'Biao', 'xxxx')", db);
    QSqlQuery query2("INSERT INTO user (id, username, password) VALUES (5, 'Bing', 'xxxx')", db);

    if (query1.lastError().type() == QSqlError::NoError &&
            query2.lastError().type() == QSqlError::NoError) {
        db.commit();   // [3] 提交事务
    } else {
        db.rollback(); // [3] 回滚事务

        qDebug() << query1.lastError().text();
        qDebug() << query2.lastError().text();
    }
}
```

调用 `transactionDemo()` 输出
> "Duplicate entry '1' for key 'PRIMARY' QMYSQL: Unable to execute query"
> " "

可以看到 query1 执行时发生了主键冲突所以插入时报错，因为数据库里已经有一条记录其 id 为 1，而 query2 执行成功，但是在数据库里却没有看到 query2 插入的用户 `Bing`，这是因为我们使用了事务，只有 query1 和 query2 同时都插入成功后才会提交事务，如果其中任意一个插入失败都会导致整个插入操作失败。

`把 query1 里的 1 改成 4`，再次调用 `transactionDemo()`，可以看到这次插入成功了。

| id   | username | password | email | mobile |
| ---- | -------- | -------- | ----- | ------ |
| 1    | Alice    | passw0rd | NULL  | NULL   |
| 2    | Bob      | Passw0rd | NULL  | NULL   |
| 3    | Josh     | Pa88w0rd | NULL  | NULL   |
| 4    | Biao     | xxxx     | NULL  | NULL   |
| 5    | Bing     | xxxx     | NULL  | NULL   |

**思考一下，我们要向 user 表里插入 100000 个用户：**

1. 开启事务每次插入 1000 个
2. 不使用事务插入 1 个

哪一个的效率高，相差大吗？

**`事物是面向数据库连接的`，思考一个问题，开启事务，更新数据库，提交|回滚事务不在同一个函数里可以吗？**

## 结束语

这一节我们主要讲了 QSqlDataBase，QSqlQuery 的常用操作，使用 Prepared Query 的方式避免 `SQL 注入攻击`提高了系统的安全性，以及介绍了事务的使用。

> `再次提示`  
> 尽可能的使用 Prepared Query 的方式来构造 SQL 语句，尤其是其中的部分内容由调用者传入，因为我们不能控制调用者传入的内容，但是我们可以控制自己的代码。

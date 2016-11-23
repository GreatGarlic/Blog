---
title: FlatBuffers 入门之 Java + Qt 版
date: 2016-11-23 14:08:14
tags: [Qt, Java]
---
FlatBuffers 是一个 Google 开源的跨平台序列化工具，支持 C, C++, C#, Go, Java, JavaScript, PHP, Python 等，好强大的感觉。

> FlatBuffers is an efficient cross platform serialization library for C++, C#, C, Go, Java, JavaScript, PHP, and Python. It was originally created at Google for game development and other performance-critical applications.

**什么时候使用 FlatBuffers?**

* FlatBuffers 的反序列化速度非常快，在交互很频繁的场景下，例如游戏服务器中它的优势非常大
* 使用 TcpSocket 通讯时序列化消息的 payload，尤其时涉及到多语言，多平台时

<!--more-->
下面 2 图为 FlatBuffers 和其他几个常用的序列化工具的比较:

![](/img/java/flatbuffers-size.jpg)
![](/img/java/flatbuffers-time.jpg)

其实 FlatBuffers 更应该和 Protobuf 比较，他们才是一个位面的工具，虽然 FlatBuffers 序列化后的数据比 Protobuf 的大那么一丢丢，运行时占用的内存是 Protobuf 的 2 倍左右，但是反序列化很速度却快很多，在硬件白菜价的今天，时间效率大多数时候更有吸引力，总体来说，更推荐 FlatBuffers。

这里主要是介绍 FlatBuffers 的入门程序，演示 Java 使用 FlatBuffers 序列化和反序列化类 Person，然后使用 Qt(C++) 反序列化 Person

1. 安装 FlatBuffers 代码生成工具
2. 下载 FlatBuffers 源码
3. 定义 Person 的 schema 文件
4. Java 使用 FlatBuffers
    1. FlatBuffers 编译 schema 文件生成 Java 的 Person 类
    2. 序列化 Person
    3. 反序列化 Person
5. Qt 使用 FlatBuffers
    1. FlatBuffers 编译 schema 文件生成 cpp 的 Person 类
    2. 反序列化 Person

## 安装 FlatBuffers 代码生成工具
需要使用 FlatBuffers 的 `flatc` 把 schema 文件生成对应的代码，例如 Java 的类:

* Mac: `brew install flatbuffers`
* Windows: 到 <https://github.com/google/flatbuffers/releases> 下载安装

## 下载 FlatBuffers 源码
使用 FlatBuffers 时引入相关的代码即可，不需要额外的库，例如 DLL 等，到 <https://github.com/google/flatbuffers> 下载

* cpp 需要用到 `include` 目录下的 `flatbuffers/flatbuffers.h`
* Java 需要用到 `java` 目录下的文件

## 定义 Person 的 schema 文件
保存为 `Person.fbs`

```
namespace com.xtuer.bean;

table Person {
    name:string;
    age:int;
}

root_type Person;
```

## Java 使用 FlatBuffers
```
├── main
│   └── java
│       └── com
│           ├── google
│           │   └── flatbuffers
│           │       ├── Constants.java
│           │       ├── FlatBufferBuilder.java
│           │       ├── Struct.java
│           │       └── Table.java
│           └── xtuer
│               └── bean
│                   └── Person.java
└── test
    └── java
        └── TestFlatBuffer.java
```

上面是项目的目录结构，下面具体介绍创建项目的过程:

1. FlatBuffers 编译 schema 文件生成 Java 的 Person 类: `flatc --java Person.fbs`，生成

    ```
    └── com
        └── xtuer
            └── bean
                └── Person.java
    ```

1. 复制上面生成的 `com` 目录到项目中  
2. 复制 FlatBuffers 源码中 `java` 目录下的 `com` 目录到项目中  
3. 创建测试用的类 TestFlatBuffer

    ```java
    import com.google.flatbuffers.FlatBufferBuilder;
    import com.xtuer.bean.Person;
    import org.junit.Test;
    
    import java.io.FileInputStream;
    import java.io.FileOutputStream;
    import java.nio.ByteBuffer;
    
    public class TestFlatBuffer {
        private static final String FILE_NAME = "/Users/Biao/Desktop/person.data";
    
        @Test
        public void testWrite() throws Exception {
            // [1] 创建 Person 对象
            FlatBufferBuilder builder = new FlatBufferBuilder();
            int orc = Person.createPerson(builder, builder.createString("道格拉斯·狗"), 30);
            builder.finish(orc);
    
            // [2] 序列化 Person 对象为字节码
            byte[] buffer = builder.sizedByteArray();
    
            // [3] 保存字节码到文件，也可以进行网络传输
            FileOutputStream out = new FileOutputStream(FILE_NAME);
            out.write(buffer);
            out.close();
        }
    
        @Test
        public void testRead() throws Exception {
            // [1] 从文件读取字节码，也可以是使用 Socket 传输得到的
            byte[] buf = new byte[1024];
            FileInputStream in = new FileInputStream(FILE_NAME);
            in.read(buf);
            in.close();
    
            // [2] 把字节码反序列化为 Person 对象
            ByteBuffer buffer = ByteBuffer.wrap(buf);
            Person person = Person.getRootAsPerson(buffer);
            System.out.println("Name: " + person.name() + ", Age: " + person.age());
        }
    }
    ```

6. 执行 `testWrite()`，序列化 Person 对象生成 `/Users/Biao/Desktop/person.data`
7. 执行 `testRead()`，反序列化得到 Person 对象，输出: 

    ```
    Name: 道格拉斯·狗, Age: 30
    ```

## Qt 使用 FlatBuffers
```
├── FlatBuffer.pro
├── Person_generated.h
├── flatbuffers
│   ├── flatbuffers.h
│   └── flatbuffers.pri
└── main.cpp
```
上面是项目的目录结构，下面具体介绍创建项目的过程:

1. FlatBuffers 编译 schema 文件生成 cpp 的 Person 类: `flatc --cpp Person.fbs`

    ```
    Person_generated.h
    ```
2. 复制上面生成的 `Person_generated.h` 到项目中
3. 复制 FlatBuffers 源码中 `include` 目录 下的 `flatbuffers/flatbuffers.h` 目录到项目中
4. 创建 main.cpp

    ```cpp
    #include "flatbuffers/flatbuffers.h"
    #include "Person_generated.h"
    
    #include <QString>
    #include <QDebug>
    #include <QFile>
    
    int main(int argc, char *argv[]) {
        Q_UNUSED(argc)
        Q_UNUSED(argv)
    
        QFile file("/Users/Biao/Desktop/person.data");
        if (!file.open(QIODevice::ReadOnly)) { return -10086; }
        QByteArray buffer = file.readAll();
    
        const com::xtuer::bean::Person *p = com::xtuer::bean::GetPerson(buffer.constData());
        qDebug() << QString("Name: %1, Age: %2").arg(QString(p->name()->c_str())).arg(p->age());
    
        qDebug() << QString::fromUtf8(p->name()->c_str());      // OK: 证明 FlatBuffer 使用的是 UTF-8 存储字符串
        qDebug() << QString::fromLocal8Bit(p->name()->c_str()); // OK: Mac 默认就是使用 UTF-8
        qDebug() << QString::fromLatin1(p->name()->c_str());    // Error: 因为不是使用 Latin1 存储字符串
    
        return 0;
    }
    ```
5. 运行程序，输出(中文也没问题，太棒了)

    ```
    "Name: 道格拉斯·狗, Age: 30"
    "道格拉斯·狗"
    "道格拉斯·狗"
    "é\u0081\u0093æ ¼æ\u008B\u0089æ\u0096¯Â·ç\u008B\u0097"
    ```

## 思考
上面的例子展示了 Java 使用 FlatBuffers 序列化对象存储到文件，然后 Qt 在从文件反序列化得到数据，或许有人要问，我还是不知道 TcpSocket 中怎么使用 FlatBuffers 传输数据啊？

其实不管像上面使用文件交互数据还是使用 TcpSocket 传输交互数据，他们交互的都是 FlatBuffers 生成的二进制数据，所以 TcpSocket 的话直接传输 FlatBuffers 生成的二进制数据就可以了，当接收到数据后用 FlatBuffers 反序列化，文件或者 TcpSocket 只是交互的方式，交互的内容都是数据，当然 TcpSocket 通讯中帧的格式，粘包等问题就不属于 FlatBuffers 的范畴了。

## 参考资料
这里只是介绍了 FlatBuffers 的入门程序，演示了怎么使用 FlatBuffers，更多的内容请参考:

* [FlatBuffers 主页](https://google.github.io/flatbuffers/index.html)
* [高效数据序列化的工具 FlatBuffers 的初体验]()
* [FlatBuffers 使用简介](http://www.jianshu.com/p/6eb04a149cd8)

















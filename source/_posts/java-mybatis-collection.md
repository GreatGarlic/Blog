---
title: MyBatis Collecton
date: 2018-01-25 14:13:16
tags: Java
---

MyBatis 中一对多的关系使用 `collection` 进行映射，但是怎么确定哪些行是同一个对象的数据呢？关键是使用 `<id>` 来归类数据(MyBatis 的文档只提到是为了提高效率)，下面介绍使用 `<id>` 和 `<result>` 的区别。

## 数据表

| name  | email           | country     | province     | street        |
| ----- | --------------- | ----------- | ------------ | ------------- |
| Biao  | biao@gmail.com  | china       | 北京           | 天河街           |
| Biao  | biao@icloud.com | Deutschland | Braunschweig | Wiesenstrasse |
| Alice | alice@gmail.com | china       | 河北           | 鼓楼大街          |

## Xml Mapper

查找所有用户

```sql
SELECT name, email, country, province, street FROM user
```

映射文件如下

```xml
<mapper namespace="com.xtuer.mapper.UserMapper">
    <select id="users" resultMap="userResultMap">
        SELECT name, email, country, province, street FROM user
    </select>

    <resultMap id="userResultMap" type="User">
        <id property="name" column="name"/> <!-- 关注这里 -->
        <result property="email" column="email"/>
        <collection property="addresses" resultMap="addressResultMap"/>
    </resultMap>

    <resultMap id="addressResultMap" type="Address">
        <result property="country" column="country"/>
        <result property="province" column="province"/>
        <result property="street" column="street"/>
    </resultMap>
</mapper>
```

<!--more-->测试用例

```java
public class UserTest {
    @Autowired
    private UserMapper mapper;

    @Test
    public void findUsers() {
        System.out.println(JSON.toJSONString(mapper.users(), true));
    }
}
```

测试输出

```json
[
    {
        "addresses": [{
                "country": "china",
                "province": "北京",
                "street": "天河街"
            },
            {
                "country": "Deutschland",
                "province": "Braunschweig",
                "street": "Wiesenstrasse"
            }
        ],
        "email": "biao@gmail.com",
        "name": "Biao"
    },
    {
        "addresses": [{
            "country": "china",
            "province": "河北",
            "street": "鼓楼大街"
        }],
        "email": "alice@gmail.com",
        "name": "Alice"
    }
]
```

上面的程序查询得到 3 条记录，有 2 条的 name 相同为 Biao 生成了一个 User 对象，name 为 Alice 的记录生成了一个 User 对象，最终程序只生成了 2 个 User 对象。

**怎么确定 name 为 Biao 的 2 行属于同一个 User 对象呢？**

因为在 `userResultMap` 中把 User 的属性 `name` 标记为 `<id>`，这样查询得到的结果中，name 相同的行都是同一个用户对象的数据(注意: 虽然 email 不同，但是只取第一行的 email，第二行的忽略)，这些记录中的每行的 country, province, street 用于构造一个 Address 对象。

如果把 `<id>` 换为 `<result>`，虽然有两行的 name 相同，但是映射时把它们作为两个不同的用户对象的数据，也就是说生成了 2 个 name 为 Biao 的 User 对象，最终程序生成了 3 个 User 对象，输出如下:

```json
[
    {
        "addresses": [{
            "country": "china",
            "province": "北京",
            "street": "天河街"
        }],
        "email": "biao@gmail.com",
        "name": "Biao"
    },
    {
        "addresses": [{
            "country": "Deutschland",
            "province": "Braunschweig",
            "street": "Wiesenstrasse"
        }],
        "email": "biao@icloud.com",
        "name": "Biao"
    },
    {
        "addresses": [{
            "country": "china",
            "province": "河北",
            "street": "鼓楼大街"
        }],
        "email": "alice@gmail.com",
        "name": "Alice"
    }
]
```

> association 中嵌套 collection，collection 中再嵌套 association 也是同理，使用 `<id>` 归类数据为某一个对象。

## Java Mapper

```java
package com.xtuer.mapper;

import com.xtuer.bean.User;
import java.util.List;

public interface UserMapper {
    List<User> users();
}
```

## Java Bean

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;
import lombok.experimental.Accessors;

import java.util.List;

@Getter
@Setter
@Accessors(chain = true)
public class User {
    private String name;
    private String email;
    private List<Address> addresses;
}
```

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;
import lombok.experimental.Accessors;

@Getter
@Setter
@Accessors(chain = true)
public class Address {
    private String country;
    private String province;
    private String street;
}
```
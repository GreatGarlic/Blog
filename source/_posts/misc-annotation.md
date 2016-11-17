---
title: Annotation - 注解
date: 2016-11-17 16:41:51
tags: Misc
---
注解的目的是给类，属性，参数等提供一些元数据信息，例如标记类的用途（都是静态的数据，写程序的时候写死在代码里）。

注解有 4 个类型，最常用的是 `@Target` 和 `@Retention`:

* **@Target** - Annotation 修饰的对象范围，即被修饰的对象可以用在什么地方，常用的有类、接口、属性和参数
    * ElementType.TYPE - 类、接口
    * ElementType.FIELD - 属性
    * ElementType.PARAMETER - 参数
* **@Retention** - 比较常用的是 RetentionPolicy.RUNTIME，运行时通过反射取得注解的元数据
* **@Documented**
* **@Inherited**

<!--more-->

Annotation 的使用步骤如下：

1. 定义 Annotation
    * Annotation 名字前面使用关键字 `@interface`
2. 使用 Annotation
    * 使用案例
    
        ```java
        @ResponseMeta(editable = true, country = Country.GERMANY)
        private String username;
        ```
    * 可以给 Annotation 赋多个值（用逗号分割），也可以不赋（全部使用默认值）
3. 用反射解析出 Annotation 的元数据
    * 判断是否用了 Annotation，使用函数 `isAnnotationPresent(Class)`
    * 取得 Annotation，使用函数 `getAnnotation(Class)`

> 最佳实践：每个 annotation 的成员都给一个业务相关的默认值，这样在使用的时候就能判断出是否使用默认值

## 1. 定义 Annotation
> * 定义 Annotation 的关键词是 `@interface`  
> * Annotation 里也可以使用 enum
> * 如果 Annotation 没有 `@Retention` 的话下面的程序就没法解析出 Annotation 的元数据

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.lang.reflect.Field;

enum Country {
    CHINA, UNITED_STATES, GERMANY
}

// [1]. 定义 annotation
@Target(ElementType.TYPE) // 作用在类上
@Retention(RetentionPolicy.RUNTIME)
@interface ScopeMeta {
}

@Target(ElementType.FIELD) // 作用在属性上
@Retention(RetentionPolicy.RUNTIME)
@interface ResponseMeta {
    public boolean editable() default false;
    public String description() default "";
    public Country country() default Country.CHINA;
}

```

## 2. 使用 Annotation

```java
@ScopeMeta
public class User {
    // [2]. 使用 annotation
    @ResponseMeta(description="User's identifier")
    private int id;

    @ResponseMeta(editable = true, country = Country.GERMANY)
    private String username;

    private String password;

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }
}
```

##3. 用反射解析出 Annotation 的元数据

```java
    public static void main(String[] args) {
        User user = new User(1, "Alice", "Passw0rd");

        // [3]. 用反射读取 annotation 内容
        if (user.getClass().isAnnotationPresent(ScopeMeta.class)) {
            System.out.println("Class User uses annotation ScopedMeta");
        }

        Field[] fields = user.getClass().getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(ResponseMeta.class)) {
                ResponseMeta meta = field.getAnnotation(ResponseMeta.class);
                System.out.printf("Field '%s' - %s\n", field.getName(), meta);
                
                if (meta.editable()) {
                    System.out.printf("Field '%s' is editable", field.getName());
                }
            }
        }
    }
```

##### 输出:
> Class User uses annotation ScopedMeta  
Field 'id' - @annotation.ResponseMeta(country=CHINA, description=User's identifier, editable=false)  
Field 'username' - @annotation.ResponseMeta(country=GERMANY, description=, editable=true)  
Field 'username' is editable

在类型 User 和 属性 id, username 上使用了 Annotation，所以输出中有它们的信息，
password 没有使用 Annotation，输出中没有 password 的信息。

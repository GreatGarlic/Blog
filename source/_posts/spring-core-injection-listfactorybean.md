---
title: ListFactoryBean 注入 List
date: 2017-04-01 15:38:39
tags: Spring-Core
---
```xml
<bean id="listExample" class="com.xtuer.beans.CollectionHolder">
   <property name="list">
       <list> <!--表示是 list-->
           <value>Gut</value>
           <value>Good</value>
           <value>没问题</value>
       </list>
   </property>
</bean>
```

上面的方式注入 List，List 的类型不受我们控制，默认是 java.util.ArrayList。ListFactoryBean 可以创建一个特定类型的 List。由于 ListFactoryBean 的配置太过麻烦，这里我们只介绍和其等效的 `<util:list>`。<!--more-->

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="listExample" class="com.xtuer.beans.CollectionHolder">
        <property name="list">
            <util:list list-class="java.util.LinkedList">
                <value>Gut</value>
                <value>Good</value>
                <value>没问题</value>
            </util:list>
        </property>
    </bean>
</beans>
```

> 1. 需要引入 util schema。
> 2. 可以不指定 `list-class`，这样就和默认的 `<list>` 注入没有什么区别了。

## ListFactoryBeanTest
```java
import com.xtuer.beans.CollectionHolder;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ListFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("listExample", CollectionHolder.class);
        CommonUtils.output(holder.getList());
        CommonUtils.output(holder.getList().getClass());
    }
}
```

## 输出
```
[
    "Gut",
    "Good",
    "没问题"
]
"java.util.LinkedList"
```
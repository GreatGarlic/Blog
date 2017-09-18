---
title: SetFactoryBean 注入 Set
date: 2017-04-01 15:42:20
tags: SpringCore
---
```xml
<bean id="setExample" class="com.xtuer.beans.CollectionHolder">
   <property name="set">
       <set> <!--表示是 set-->
           <value>Gut</value>
           <value>Good</value>
           <value>没问题</value>
       </set>
   </property>
</bean>
```

上面的方式注入 Set，Set 的类型不受我们控制，默认是 java.util.LinkedHashSet。SetFactoryBean 可以创建一个特定类型的 Set。由于 SetFactoryBean 的配置太过麻烦，这里我们只结束和其等效的 `<util:set>`。<!--more-->

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

    <bean id="setExample" class="com.xtuer.beans.CollectionHolder">
        <property name="set">
            <util:set set-class="java.util.HashSet">
                <value>Gut</value>
                <value>Good</value>
                <value>没问题</value>
            </util:set>
        </property>
    </bean>
</beans>
```

> 1. 需要引入 util schema。
> 2. 可以不指定 `set-class`，这样就和默认的 `<set>` 注入没有什么区别了。

## SetFactoryBeanTest
```java
import com.xtuer.beans.CollectionHolder;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SetFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("setExample", CollectionHolder.class);
        CommonUtils.output(holder.getSet());
        CommonUtils.output(holder.getSet().getClass());
    }
}
```

## 输出
```
[
    "没问题",
    "Gut",
    "Good"
]
"java.util.HashSet"
```
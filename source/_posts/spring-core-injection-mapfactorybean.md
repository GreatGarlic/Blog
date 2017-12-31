---
title: MapFactoryBean 注入 Map
date: 2017-04-01 15:44:15
tags: SpringCore
---

```xml
<bean id="mapExample" class="com.xtuer.beans.CollectionHolder">
   <property name="map">
       <map> <!--表示是 map-->
           <entry key="German" value="Gut"/>
           <entry key="English" value="Good"/>
           <entry key="Chinese" value="没问题"/>
       </map>
   </property>
</bean>
```

上面的方式注入 Map，Map 的类型不受我们控制，默认是 java.util.LinkedHashMap。MapFactoryBean 可以创建一个特定类型的 Map。由于 MapFactoryBean 的配置太过麻烦，这里我们只结束和其等效的 `<util:map>`。<!--more-->

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

    <bean id="mapExample" class="com.xtuer.beans.CollectionHolder">
        <property name="map">
            <util:map map-class="java.util.HashMap">
                <entry key="German"  value="Gut"/>
                <entry key="English" value="Good"/>
                <entry key="Chinese" value="没问题"/>
            </util:map>
        </property>
    </bean>
</beans>
```

> 1. 需要引入 util schema。
> 2. 可以不指定 `map-class`，这样就和默认的 `<map>` 注入没有什么区别了。

## MapFactoryBeanTest
```java
import com.xtuer.beans.CollectionHolder;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MapFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("mapExample", CollectionHolder.class);
        CommonUtils.output(holder.getMap());
        CommonUtils.output(holder.getMap().getClass());
    }
}
```

## 输出
```
{
    "English" : "Good",
    "Chinese" : "没问题",
    "German" : "Gut"
}
"java.util.HashMap"
```

## 注入静态变量

使用 `util:constant`

```xml
<property name="classSerializers">
    <map>
        <!-- key 时 Class, value 时静态变量 -->
        <entry key="java.lang.Long">
            <util:constant static-field="com.alibaba.fastjson.serializer.ToStringSerializer.instance"/>
        </entry>
    </map>
</property>
```


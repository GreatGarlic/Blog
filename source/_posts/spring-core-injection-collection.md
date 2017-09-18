---
title: List, Set, Map, Properties 注入
date: 2017-04-01 15:34:03
tags: SpringCore
---
## 注入 List
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

`<list>` 里不只是可以使用 `<value>`，还可以使用 `<ref bean="">`，`<bean class="ClassName">`，如下

```xml
<bean id="listExample" class="com.xtuer.beans.CollectionHolder">
   <property name="list">
       <list> <!--表示是 list-->
           <value>Gut</value>
           <value>Good</value>
           <value>没问题</value>
           
           <ref bean="user"/>
           <bean class="com.xtuer.beans.User"/>
       </list>
   </property>
</bean>
```

<!--more-->

## 注入数组

数组的注入和 List 完全一样

## 注入 Set
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

Set 的注入和 List 一样，只是把 `<list>` 换成 `<set>`

## 注入 Map
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

`<entry>` 里还可以是 `<value-ref>` 和 `<bean>`，如下

```xml
<bean id="mapExample" class="com.xtuer.beans.CollectionHolder">
   <property name="map">
       <map> <!--表示是 map-->
           <entry key="German" value="Gut"/>
           <entry key="English" value="Good"/>
           <entry key="Chinese" value="没问题"/>

           <entry key="user" value-ref="user"/>
           <entry key="customer">
               <bean class="com.xtuer.beans.Customer">
                   <constructor-arg value="Alice"/>
                   <constructor-arg value="40"/>
               </bean>
           </entry>
       </map>
   </property>
</bean>
```

## 注入 Properties
```xml
<bean id="propsExample" class="com.xtuer.beans.CollectionHolder">
   <property name="props">
       <props> <!--表示是 properties-->
           <prop key="German">Gut</prop> <!--只能是字符串，不能是 ref, bean-->
           <prop key="English">Good</prop>
           <prop key="Chinese">没问题</prop>
       </props>
   </property>
</bean>
```

由于 Properties 的 value 只能是字符串，所以 `<prop>` 的值只能是字符串，不能是 `<ref>`，`<bean>`

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.xtuer.beans.User"/>

    <bean id="arrayExample" class="com.xtuer.beans.CollectionHolder">
        <property name="array">
            <list>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </list>
        </property>
    </bean>
    
    <bean id="listExample" class="com.xtuer.beans.CollectionHolder">
        <property name="list">
            <list>
                <value>Gut</value>
                <value>Good</value>
                <value>没问题</value>
                <ref bean="user"/>
                <bean class="com.xtuer.beans.User"/>
            </list>
        </property>
    </bean>

    <bean id="setExample" class="com.xtuer.beans.CollectionHolder">
        <property name="set">
            <set>
                <value>Gut</value>
                <value>Good</value>
                <value>没问题</value>
            </set>
        </property>
    </bean>

    <bean id="mapExample" class="com.xtuer.beans.CollectionHolder">
        <property name="map">
            <map>
                <entry key="German" value="Gut"/>
                <entry key="English" value="Good"/>
                <entry key="Chinese" value="没问题"/>

                <entry key="user" value-ref="user"/>
                <entry key="customer">
                    <bean class="com.xtuer.beans.Customer">
                        <constructor-arg value="Alice"/>
                        <constructor-arg value="40"/>
                    </bean>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="propsExample" class="com.xtuer.beans.CollectionHolder">
        <property name="props">
            <props>
                <prop key="German">Gut</prop>
                <prop key="English">Good</prop>
                <prop key="Chinese">没问题</prop>
            </props>
        </property>
    </bean>
</beans>
```

## CollectionHolder

CollectionHolder 的属性有 list, set, map, properties

```java
package com.xtuer.beans;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

public class CollectionHolder {
    private List<Object> list;
    private Set<Object> set;
    private Map<String, Object> map;
    private Properties props;
    private int[] array;

    public List<Object> getList() {
        return list;
    }

    public void setList(List<Object> list) {
        this.list = list;
    }

    public Set<Object> getSet() {
        return set;
    }

    public void setSet(Set<Object> set) {
        this.set = set;
    }

    public Map<String, Object> getMap() {
        return map;
    }

    public void setMap(Map<String, Object> map) {
        this.map = map;
    }

    public Properties getProps() {
        return props;
    }

    public void setProps(Properties props) {
        this.props = props;
    }

    public int[] getArray() {
        return array;
    }

    public void setArray(int[] array) {
        this.array = array;
    }
}
```

## CollectionTest

```java
import com.xtuer.beans.CollectionHolder;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class CollectionTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }
    
    @Test
    public void testArray() {
        CollectionHolder holder = context.getBean("arrayExample", CollectionHolder.class);
        CommonUtils.output(holder.getArray());
        CommonUtils.output(holder.getArray().getClass());
    }

    @Test
    public void testList() {
        CollectionHolder holder = context.getBean("listExample", CollectionHolder.class);
        CommonUtils.output(holder.getList());
        CommonUtils.output(holder.getList().getClass());
    }

    @Test
    public void testSet() {
        CollectionHolder holder = context.getBean("setExample", CollectionHolder.class);
        CommonUtils.output(holder.getSet());
        CommonUtils.output(holder.getSet().getClass());
    }

    @Test
    public void testMap() {
        CollectionHolder holder = context.getBean("mapExample", CollectionHolder.class);
        CommonUtils.output(holder.getMap());
        CommonUtils.output(holder.getMap().getClass());
    }

    @Test
    public void testProps() {
        CollectionHolder holder = context.getBean("propsExample", CollectionHolder.class);
        CommonUtils.output(holder.getProps());
        CommonUtils.output(holder.getProps().getClass());
    }
}
```
---
title: Spring Security 权限继承
date: 2017-08-29 16:34:07
tags: SpringSecurity
---

如下面层级结构的权限:
```
ROLE_ADMIN > ROLE_STAFF
ROLE_STAFF > ROLE_USER
ROLE_USER  > ROLE_GUEST
```

A user who is authenticated with `ROLE_ADMIN`, will behave as if they have all four roles, as well the user will have the authorities `ROLE_USER` and `ROLE_GUEST` who is authenticated with `ROLE_USER`，[请参考帮助文档](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#authz-hierarchical-roles)。

Spring Security 里通过 `RoleHierarchyVoter` 实现权限继承(只需要配置，不需要写代码)。

> **注意:** 
>
> `<http>` 需要设置 use-expressions 为 false，禁用 SpEL 表达式进行权限判断，因为它为 true 时 Spring Security 使用 WebExpressionConfigAttribute，它的 getAttribute() 总是返回 null，导致 RoleVoter.supports() 总是返回 false，于是权限校验失败，为 false 时使用的是 SecurityConfig，这时就没问题了，可以在 RoleVoter.vote() 中打断点进行验证。
>
> 坑爹的是，Spring Security 的帮助文档里没说这个，走了不少弯路。<!--more-->

## 实现权限继承，需要如下配置

* 不使用 SpEL 表达式: **use-expressions="false"**
* 使用自定义的 AccessDecisionManager: **access-decision-manager-ref="accessDecisionManager"**
* 创建需要的 bean: **accessDecisionManager, roleVoter, roleHierarchy**
* 在 roleHierarchy 中配置层级的权限, `a > b` 表示 a 继承 b，b 有的权限 a 都有，权限的继承可以传递

## 参考配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
        xmlns="http://www.springframework.org/schema/security"
        xmlns:beans="http://www.springframework.org/schema/beans"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd">
    <context:annotation-config/>

    <http security="none" pattern="/page/login"/>
    <http security="none" pattern="/static/**"/>

    <http auto-config="true" use-expressions="false" access-decision-manager-ref="accessDecisionManager">
        <intercept-url pattern="/page/admin"   access="ROLE_ADMIN"/>
        <intercept-url pattern="/demo/filters" access="ROLE_USER"/>
        ...
    </http>
    ...

    <!-- 自己定义 AccessDecisionManager 对应的 bean，实现角色继承 -->
    <beans:bean id="accessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
        <beans:constructor-arg>
            <beans:list>
                <beans:ref bean="roleVoter"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
    <beans:bean id="roleVoter" class="org.springframework.security.access.vote.RoleHierarchyVoter">
    	<beans:constructor-arg ref="roleHierarchy"/>
    </beans:bean>
    <beans:bean id="roleHierarchy" class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
    	<beans:property name="hierarchy">
    		<beans:value>
                ROLE_ADMIN > ROLE_USER
                ROLE_STAFF > ROLE_USER
                ROLE_USER  > ROLE_GUEST
    		</beans:value>
    	</beans:property>
    </beans:bean>
</beans:beans>
```

## 思考

如果我们自己实现权限的继承，那么就需要实现一个权限映射的函数，获取用户信息的时候进行转换，可参考 RoleHierarchyImpl 的实现。
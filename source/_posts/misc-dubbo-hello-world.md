---
title: Dubbo Hello World
date: 2017-04-12 07:23:38
tags: [Java, Misc]
---

本文介绍 Dubbo 的入门程序:

* 注册和查找服务使用 ZooKeeper
  * ZooKeeper 的安装请参考 [本机安装 ZooKeeper 集群](/misc-zookeeper)


* Dubbo Provider: Dubbo + SpringMvc
* Dubbo Consumer: Dubbo + JUnit 

> 使用传统 RPC 时服务器之间是星形结构，紧耦合的，Dubbo (SOA) 时服务器之间通过调度中心调度，典型的中介者模式，实现了解耦，服务的注册和查找通过中心的 ZooKeeper 使用接口实现，不需要配置 URL，服务器之间互相不知道对方的存在。Spring Http Remote Invoker 也能实现远程方法调用，但是需要配置  URL，而且也是星形结构。<!--more-->

下面的内容主要摘抄自 [Dubbo 用户指南](http://dubbo.io/User+Guide-zh.htm)

## 背景

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

![img](/img/misc/dubbo-architecture-roadmap.jpg)

* **单一应用架构**当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的 **数据访问框架(ORM)** 是关键。
* **垂直应用架构**当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的 **Web框架(MVC)** 是关键。
* **分布式服务架构**当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的 **分布式服务框架(RPC)** 是关键。
* **流动计算架构**当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个**调度中心**基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的 **资源调度和治理中心(SOA)** 是关键。

## 架构

![img](/img/misc/dubbo-architecture.jpg)

**节点角色说明：**

* **Provider:** 暴露服务的服务提供方。
* **Consumer:** 调用远程服务的服务消费方。
* **Registry:** 服务注册与发现的注册中心。
* **Monitor:** 统计服务的调用次调和调用时间的监控中心。
* **Container:** 服务运行容器。

**调用关系说明：**

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

**(1) 连通性：**

* 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
* 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
* 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
* 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
* 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
* 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
* 注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
* 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

**(2) 健状性：**

* 监控中心宕掉不影响使用，只是丢失部分采样数据
* 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
* 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
* 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
* 服务提供者无状态，任意一台宕掉后，不影响使用
* 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

**(3) 伸缩性：**

* 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
* 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

**(4) 升级性：**

* 当服务集群规模进一步扩大，带动 IT 治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力：

![img](/img/misc/dubbo-architecture-future.jpg)

* ~~**Deployer:** 自动部署服务的本地代理。~~
* ~~**Repository:** 仓库用于存储服务应用发布包。~~
* ~~**Scheduler:** 调度中心基于访问压力自动增减服务提供者。~~
* ~~**Admin:** 统一管理控制台。~~

## 一、启动 ZooKeeper

执行下面的命令启动 ZooKeeper 服务器:

* /Users/Biao/Documents/zookeeper/zookeeper-1/bin/zkServer.sh start
* /Users/Biao/Documents/zookeeper/zookeeper-2/bin/zkServer.sh start
* /Users/Biao/Documents/zookeeper/zookeeper-3/bin/zkServer.sh start

使用命令 **jps** 可以看到 ZooKeeper 的进程信息

```
17145 QuorumPeerMain
17212 QuorumPeerMain
17234 QuorumPeerMain
```

或则从任务管理器里可以看到 3 个名为 java 的进程，看看进程信息就知道是不是 ZooKeeper 的进程了，很多 Java 程序的进程名都叫 java。

> 参考 [启动-ZooKeeper](/misc-zookeeper/#启动-ZooKeeper)

## 二、Gradle 依赖

```groovy
compile('com.alibaba:dubbo:2.5.3') {
    exclude group: 'org.springframework', module: 'spring'
}
compile('com.101tec:zkclient:0.10')

testCompile("org.springframework:spring-test:$versions.spring")
testCompile('junit:junit:4.12')
```

## 三、Dubbo Provider

定义提供服务的接口 TimeService 和其实现 TimeServiceImpl:

```java
package com.xtuer.dubbo;

public interface TimeService {
    String getTime();
    String getTime(String s);
}
```

```java
package com.xtuer.dubbo;

public class TimeServiceImpl implements TimeService {
    @Override
    public String getTime() {
        return "Time: " + System.currentTimeMillis();
    }

    @Override
    public String getTime(String s) {
        return String.format("Time - %s -: %d", s, System.currentTimeMillis());
    }
}
```

Dubbo Provider 配置文件 **spring-dubbo-provider.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="provider-of-hello-world-app"  />

    <dubbo:registry id="dubbo-registry" address="zookeeper://127.0.0.1:2181" file="/temp/dubbo.cachr" />

    <!-- 用 dubbo 协议在 20880 端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />

    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.xtuer.dubbo.TimeService" ref="timeService" />

    <!-- 和本地bean一样实现服务 -->
    <bean id="timeService" class="com.xtuer.dubbo.TimeServiceImpl" />
</beans>
```

把 spring-dubbo-provider.xml 加入到 SpringMvc 的配置文件里即可，然后启动项目。

> 通过接口 **com.xtuer.dubbo.TimeService** 来注册和查找服务。
>
> 上面的 Dubbo Provider 放在了 Web 项目里，也可以使用 JUnit，可执行程序等来作为服务运行，这样需要的系统资源就会少很多，不一定要在 Web 项目里使用。

## 四、Dubbo Consumer

复制上面提供服务的接口 TimeService 到项目中，然后配置 Dubbo Consumer 到 spring-dubbo-consumer.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-hello-world-app"/>

    <dubbo:registry id="dubbo-registry" address="zookeeper://127.0.0.1:2181" file="/temp/dubbo.cachr" />

    <!-- 生成远程服务代理，可以和本地 bean 一样使用 timeService -->
    <dubbo:reference id="timeService" interface="com.xtuer.dubbo.TimeService"/>
</beans>
```

使用下面的类测试访问 Dubbo Provider 提供的时间服务:

```java
import com.xtuer.dubbo.TimeService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@ContextConfiguration({"classpath:config/spring-dubbo-consumer.xml"})
public class DubboConsumerTest {
    @Autowired
    private TimeService timeService;

    @Test
    public void testRemote() {
        // 远程调用访问 Dubbo 提供的服务
        System.out.println(timeService.getTime());
        System.out.println(timeService.getTime("Lion"));
    }
}
```

输出如:

```
Time: 1491954054411
Time - Lion -: 1491954054422
```


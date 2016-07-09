---
title: Spring 判断设备信息
date: 2016-07-09 21:32:04
tags: [Spring, Java]
---

虽然响应式设计现在很流行，设计的好的话同一个页面在桌面设备和移动设备都能显示的很好，但是源码都一样的，也就是说桌面设备和移动设备要下载的文件都是一样大小的，在用户体验，页面显示效果和加载速度上不是非常理想，所以有必要针对移动设备进行优化，例如 jQuery 在移动设备上应该使用 jQuery Mobile，图片进行相应的缩小，很多桌面端的内容不应该出现在移动设备中，例如广告，侧边栏，复杂的搜索，动态获取的第三方信息等。

为了检测页面访问的设备类型，可以使用 `Spring Mobile`，当然也可以自己读取 `User Agent` 来判断。

<!--more-->

## Gradle 依赖
```groovy
compile('org.springframework.mobile:spring-mobile-device:1.1.5.RELEASE')
```

## 配置拦截器
```xml
<mvc:interceptors>
    <bean class="org.springframework.mobile.device.DeviceResolverHandlerInterceptor"/>
</mvc:interceptors>
```

## 配置 `annotation-driven`
```xml
<mvc:annotation-driven>
    <mvc:argument-resolvers>
        <bean class="org.springframework.mobile.device.DeviceWebArgumentResolver"/>
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```

## 获取设备类型
```java
@GetMapping("/device")
@ResponseBody
public String detectDevice(Device device) {
    if (device.isMobile()) {
        return "Mobile";
    } else if (device.isTablet()) {
        return "Tablet";
    } else {
        return "Desktop";
    }
}
```

> 访问同一个 URL，对于不同的设备，返回不同的 View，针对设备进行代码优化，例如
```java
@GetMapping("/help")
public String help(Device device) {
    if (device.isMobile()) {
        return "m_help.html";
    } else {
        return "help.html";
    }
}
```
>
> 如果要考虑 SEO 的话，那就同一个页面不同的设备使用不同的代码，并且 URL 也不一样，例如 http://www.xtuer.com/help 和 http://www.xtuer.com/m/help

## 参考文档
* [Spring Mobile Device Module](http://docs.spring.io/spring-mobile/docs/current/reference/html/device.html)

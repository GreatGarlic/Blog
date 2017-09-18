---
title: SpringMVC 获取 Request 和 Response
date: 2016-10-07 12:09:35
tags: SpringMVC
---

SpringMVC 中在任意地方取得 `HttpServletRequest` 和 `HttpServletResponse`

1. 在 web.xml 中注册 `RequestContextListener` (SpringMVC 4 不需要这一步)

    ```xml
    <listener>  
        <listener-class>  
            org.springframework.web.context.request.RequestContextListener  
        </listener-class>  
    </listener>
    ```

2. 获取 `HttpServletRequest` 和 `HttpServletResponse`

    ```java
        public static String testRequestAndResponse() {
            HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
            HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();
            
            return request.getParameter("name");
        }
    ```

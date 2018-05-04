---
title: SpringMVC 使用 @PathVariable 获取有 . 的 URL 中的变量
date: 2018-05-04 10:07:12
tags: SpringMVC
---

```java
@GetMapping("/api/file/{filename}")
@ResponseBody
public String foo(@PathVariable String filename) {
    return filename;
}
```

默认配置时，不同的 URL ，获取到的 filename 为:

* `/api/file/foo`，filename 为 foo
* `/api/file/foo.pdf`，filename 为 foo
* `/api/file/foo.pdf.png`，filename 为 foo.pdf
* `/api/file/foo.pdf.png.doc`，filename 为 foo.pdf.png

最后一个 `.` 被截断了，解决这个问题有 2 中方法:

* 使用正则表达式进行路径匹配，映射为：`@GetMapping("/api/file/{filename:.+}")`

  * 缺点：每个路径的映射都要写一遍，不方便
  * 优点：缺点也是优点，只影响需要的路径

* 配置 `annotation-driven`，映射为：`@GetMapping("/api/file/{filename}")`

  * 优点：只需要配置一次，整个应用都生效，方便
  * 缺点：优点也是缺点，影响了整个系统，不过还没有发现对整个系统有什么副作用

  ```xml
  <mvc:annotation-driven>
      <mvc:path-matching registered-suffixes-only="true"/>
      ...
  </mvc:annotation-driven>
  ```

<!--more-->

不过还没完，不管采用上面的哪一种方式，虽然能够正确的获取到了路径中的变量，但是如果返回的结果不是字符串，而是一个对象(已经配置好使用 FastJSON 转换为 JSON)，会抛出 406 错误:

> The resource identified by this request is only capable of generating responses with characteristics not acceptable according to the request "accept" headers.

参考下面的请求处理：

```java
@GetMapping("/api/file/{filename}")
@ResponseBody
public Result foo(@PathVariable String filename) {
    return Result.ok(filename);
}
```

网上给出了很多办法，例如配置 annotation-driven 的 content-negotiation-manager 等，尝试了很多方案结果都不行，最后发现直接把响应写入 response 能解决这个问题：

```java
@GetMapping("/api/file/{filename}")
@ResponseBody
public void foo(@PathVariable String filename, HttpServletResponse response) {
    ajaxResponse(response, filename);
}

public static void ajaxResponse(HttpServletResponse response, String data) {
    response.setContentType("application/json"); // 使用 ajax 的方式
    response.setCharacterEncoding("UTF-8");
    response.setStatus(200);

    try {
        // 写入数据到流里，刷新并关闭流
        PrintWriter writer = response.getWriter();
        writer.write(data);
        writer.flush();
        writer.close();
    } catch (IOException ex) {
    }
}
```

至于为什么会发生 406 这个异常，可以参考下 [Spring MVC + JSON = 406 Not Acceptable](https://my.oschina.net/xiaohui249/blog/604064)。
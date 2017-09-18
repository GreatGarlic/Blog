---
title: Spring 上传文件
date: 2016-04-27 22:13:10
tags: [SpringMVC, Java, Util]
---

SpringMVC 中使用 `commons-fileupload` 上传文件，需要在 SpringMVC 的配置文件里先配置 `multipartResolver`，然后就可以使用 `MultipartFile` 读取 HTTP 请求中的文件流并保存到本地了。

<!--more-->

## Gradle 依赖
```groovy
"commons-fileupload:commons-fileupload:1.3.1"
```

## 配置 multipartResolver
需要在 SpringMVC 的配置文件中配置 `multipartResolver` 用于上传文件。

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

## 上传单个文件
```java
package com.xtuer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.FileCopyUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.ServletContext;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

@Controller
public class UploadController {
    @Autowired
    private ServletContext servletContext;

    /**
     * 上传单个文件的页面
     * @return 页面的路径
     */
    @RequestMapping(value = "/upload-file", method = RequestMethod.GET)
    public String uploadFilePage() {
        return "upload-file.html";
    }

    /**
     * 上传单个文件
     *
     * @param file 上传文件 MultipartFile 的对象
     * @return 上传的结果
     */
    @RequestMapping(value = "/upload-file", method = RequestMethod.POST)
    @ResponseBody
    public String uploadFile(@RequestParam("file") MultipartFile file) {
        saveFile(file);

        return "Success";
    }

    /**
     * 把 HTTP 请求中的文件流保存到本地
     *
     * @param file MultipartFile 的对象
     */
    private boolean saveFile(MultipartFile file) {
        if (!file.isEmpty()) {
            try {
                // getRealPath() 取得 WEB-INF 所在文件夹路径
                // 如果参数是 "/temp", 当 temp 存在时返回 temp 的本地路径, 不存在时返回 null/temp (无效路径)
                String path = servletContext.getRealPath("") + File.separator + file.getOriginalFilename();
                FileCopyUtils.copy(file.getInputStream(), new FileOutputStream(path));

                return true;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return false;
    }
}
```

## upload-file.html
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body style="padding: 50px">
    选择文件
    <form action="/upload-file" method="post" enctype="multipart/form-data">
        <input type="file" name="file"/><br>
        <button type="submit">上传</button>
    </form>
</body>
</html>
```

## 测试上传单个文件
1. 访问 <http://localhost:8080/upload-file>
2. 选择文件，点击 `上传` 按钮
3. 在 WEB-INF 所在目录中查看文件是否已经上传好了

## 上传多个文件
上传多个文件和上传单个文件没多大区别，不通的只是映射函数中的参数不是 MultipartFile，而是 `MultipartFile 的数组`。

```java
    /**
     * 上传多个文件的页面
     * @return 页面的路径
     */
    @RequestMapping(value = "/upload-files", method = RequestMethod.GET)
    public String uploadFilesPage() {
        return "upload-files.html";
    }

    /**
     * 上传多个文件
     *
     * @param files 上传文件 MultipartFile 的对象数组
     * @return 上传的结果
     */
    @RequestMapping(value = "/upload-files", method = RequestMethod.POST)
    @ResponseBody
    public String uploadFiles(@RequestParam("files") MultipartFile[] files) {
        for (MultipartFile file : files) {
            saveFile(file);
        }

        return "Success";
    }
```

## upload-files.html
表单中也只是多了几个 `<input type="file" name="file"/>`，名字都是 `file`，这样表单提交时多个同名的参数就表示数组。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body style="padding: 50px">
选择文件
<form action="/upload-files" method="post" enctype="multipart/form-data">
    <input type="file" name="files"/><br>
    <input type="file" name="files"/><br>
    <input type="file" name="files"/><br>
    <button type="submit">上传</button>
</form>
</body>
</html>
```

## 测试上传多个文件
1. 访问 <http://localhost:8080/upload-files>
2. 选择多个文件，点击 `上传` 按钮
3. 在 WEB-INF 所在目录中查看文件是否已经上传好了

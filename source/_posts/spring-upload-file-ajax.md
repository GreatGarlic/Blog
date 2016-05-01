---
title: Spring Ajax 拖拽上传文件
date: 2016-05-01 17:02:34
tags: [Spring, Java, Util, Ajax]
---

在 [Spring 上传文件](http://xtuer.github.io/spring-upload-file/) 介绍了在 SpringMVC 中怎么上传文件，这里将介绍 SpringMVC ＋ Ajax ＋ 拖拽进行文件的上传，使用的是 jQuery 的文件上传插件 `jQuery File Upload`，其主页为 <http://plugins.jquery.com/blueimp-file-upload/>

<!--more-->

实现的效果如图:

![](/img/spring/ajax-upload-file.png)

* 可以点击 `Select files...` 弹出文件选择对话框选择文件上传，可以选择多个
* 拖拽文件到 `Drop files here` 进行上传
* 在 JS 中限制了文件最大为 1M，超出 1M 的不会被上传
* 在 JS 中限制了文件的类型为 jpg, png, gif (大小写不敏感)，其他类型的文件不会被上传
* 文件上传成功后会显示出其信息

## 服务器端实现
服务器端的实现和 [Spring 上传文件](http://xtuer.github.io/spring-upload-file/) 中的实现没有什么区别，只不过为了更友好的响应文件上传成功的信息给浏览器端，返回了更多的响应信息

```java
package com.xtuer.controller;

import com.xtuer.bean.FileMeta;
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
import java.util.LinkedList;

@Controller
public class UploadController {
    @Autowired
    private ServletContext servletContext;

    /**
     * Ajax 上传文件的页面
     *
     * @return 页面的路径
     */
    @RequestMapping(value = "/ajax-upload-files", method = RequestMethod.GET)
    public String ajaxUploadFilesPage() {
        return "ajax-upload-files.html";
    }

    /**
     *
     * @param files 上传文件 MultipartFile 的对象数组
     * @return 成功上传文件的信息, [{"fileName":"app_engine-85x77.png","fileSize":"8 Kb","fileType":"image/png"}, ...]
     */
    @RequestMapping(value="/ajax-upload-files", method = RequestMethod.POST)
    @ResponseBody
    public LinkedList<FileMeta> ajaxUploadFiles(@RequestParam("files") MultipartFile[] files) {
        LinkedList<FileMeta> uploadedFiles = new LinkedList<FileMeta>();

        for (MultipartFile file : files) {
            if (saveFile(file)) {
                FileMeta fileMeta = new FileMeta();
                fileMeta.setFileName(file.getOriginalFilename());
                fileMeta.setFileSize(file.getSize()/1024 + " Kb");
                fileMeta.setFileType(file.getContentType());

                uploadedFiles.add(fileMeta);
            }
        }

        return uploadedFiles;
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

## 上传文件的页面
### 需要的 CSS
```html
    <!-- Bootstrap styles 可选 -->
    <link rel="stylesheet" href="/js/bootstrap/css/bootstrap.min.css">
    <!-- CSS to style the file input field as button -->
    <link rel="stylesheet" href="/css/jquery.fileupload.css">
    <link rel="stylesheet" href="/css/dropzone.css">
```

### 需要的 JS
```html
    <script src="/js/jquery.min.js"></script>
    <!-- The jQuery UI widget factory, can be omitted if jQuery UI is already included -->
    <script src="/js/jquery.ui.widget.js"></script>
    <!-- The Iframe Transport is required for browsers without support for XHR file uploads -->
    <script src="/js/jquery.iframe-transport.js"></script>
    <!-- The basic File Upload plugin -->
    <script src="/js/jquery.fileupload.js"></script>
    <!-- 验证文件类型，大小等使用 -->
    <script src="/js/jquery.fileupload-process.js"></script>
    <script src="/js/jquery.fileupload-validate.js"></script>
```

如果不需要验证文件大小和类型，则不需要引入这 2 个 JS

> `<script src="/js/jquery.fileupload-process.js"></script>`  
> `<script src="/js/jquery.fileupload-validate.js"></script>`  
> 注意: 他们必须在 jquery.fileupload.js 后面引用

### 上传文件的页面 ajax-file-upload.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>jQuery File Upload Demo - Basic version</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Bootstrap styles 可选 -->
    <link rel="stylesheet" href="/js/bootstrap/css/bootstrap.min.css">
    <!-- CSS to style the file input field as button -->
    <link rel="stylesheet" href="/css/jquery.fileupload.css">
    <link rel="stylesheet" href="/css/dropzone.css">

    <style>
    .fileinput-button {
        width: 150px;
    }
    </style>
</head>

<body style="padding: 50px">
    <!-- The fileinput-button span is used to style the file input field as button -->
    <span class="btn btn-success fileinput-button">
        <i class="glyphicon glyphicon-plus"></i>
        <span>Select files...</span>
        <!-- The file input field used as target for the file upload widget -->
        <input id="fileupload" type="file" name="files" multiple>
    </span>
    <div id="dropzone">Drop files here</div>
    <!-- The container for the uploaded files -->
    <div id="uploaded-files" class="uploaded-files"></div>

    <!-------------------------- JS -------------------------->
    <script src="/js/jquery.min.js"></script>
    <!-- The jQuery UI widget factory, can be omitted if jQuery UI is already included -->
    <script src="/js/jquery.ui.widget.js"></script>
    <!-- The Iframe Transport is required for browsers without support for XHR file uploads -->
    <script src="/js/jquery.iframe-transport.js"></script>
    <!-- The basic File Upload plugin -->
    <script src="/js/jquery.fileupload.js"></script>
    <!-- 验证文件类型，大小等使用 -->
    <script src="/js/jquery.fileupload-process.js"></script>
    <script src="/js/jquery.fileupload-validate.js"></script>

    <script>
    $(function () {
        var url = '/ajax-upload-files';
        $('#fileupload').fileupload({
            url: url,
            dataType: 'json',
            maxFileSize: 1024000, // 允许上传的文件的最大大小 (1M)
            acceptFileTypes: /(\.|\/)(gif|jpe?g|png)$/i, // 允许上传的文件类型
            dropZone: $('#dropzone'), // 限制拖拽到 dropzone 里才能上传文件, 如果没有这个参数, 拖拽到浏览器里任何地方就上传了
            done: function(e, data) {
                // 显示上传的结果
                $.each(data.result, function(index, file) {
                    $('<p/>').text(file.fileName).appendTo('#uploaded-files');
                });
            }
        }).on('fileuploadprocessalways', function (e, data) {
            if (data.files.error) {
                alert(data.files[0].error); // 如果上传的文件验证不通过, 显示错误信息
            } else {
                data.submit(); // 执行上传操作, 上传多个文件时, 每个文件执行一次单独的上传
            }
        });

        enableDropEffect(); // 可选
    });

    // 当拖拽文件到 dropzone 上时给其添加效果
    function enableDropEffect() {
        $('#dropzone').bind('dragover', function(e) {
            var dropZone = $('#dropzone');
            var timeout = window.dropZoneTimeout;

            if (!timeout) {
                dropZone.addClass('in');
            } else {
                clearTimeout(timeout);
            }

            var found = false;
            var node = e.target;

            do {
                if (node === dropZone[0]) {
                    found = true;
                    break;
                }
                node = node.parentNode;
            } while (node != null);

            if (found) {
                dropZone.addClass('hover');
            } else {
                dropZone.removeClass('hover');
            }

            window.dropZoneTimeout = setTimeout(function () {
                window.dropZoneTimeout = null;
                dropZone.removeClass('in hover');
            }, 100);
        });
    }
    </script>
</body>
</html>
```

## FileMeta
```java
package com.xtuer.bean;

public class FileMeta {
    private String fileName;
    private String fileSize;
    private String fileType;

    // Setters and getters
}
```

## 参考
* [jQuery File Upload](http://plugins.jquery.com/blueimp-file-upload/)
* [jQuery File Upload Document](https://github.com/blueimp/jQuery-File-Upload/wiki)
* [jQuery File Upload Options](https://github.com/blueimp/jQuery-File-Upload/wiki/Options)
* [源码](/download/file-upload.7z)

---
title: HttpServletResponse 下载文件
date: 2017-05-22 11:11:41
tags: [Java, Util]
---

实现点击按钮下载文件以及点击 a 标签下载文件，注意一下几个问题:

* 浏览器中点击链接下载文件没啥好说的，但是点击按钮怎么实现下载呢？

  ```html
  调用 window.open(url) 就可以了
  ```

* 服务器端需要设置响应头表明是以流的形式下载文件

  ```java
  response.setContentType("application/octet-stream");
  ```

* 文件名有中文时需要处理乱码问题

  ```java
  String filename = new String(paper.getOriginalName().getBytes("UTF-8"), "ISO8859_1"); // 解决乱码问题
  response.setHeader("Content-Disposition", "attachment;filename=" + filename);
  ```

  <!--more-->

## 浏览器端

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
</head>

<body>
    <a href="/download/papers/1">点击链接下载</a>
    <button onclick="download1()">点击按钮下载</button>
  	<button onclick="download2">点击按钮下载</button>

    <script>
        // 会打开一个空白页下载，然后空白页消失，用户体验不好
        function download1() {
            window.open('/download/papers/1');
        }

        // 直接下载，用户体验好
        function download2() {
            var $form = $('<form method="GET"></form>');
            $form.attr('action', '/download/papers/1');
            $form.appendTo($('body'));
            $form.submit();
        }
    </script>
</body>

</html>
```

## 服务器端

```java
@GetMapping("/download/papers/{paperId}")
public void downloadPaper(@PathVariable String paperId, HttpServletResponse response) {
    InputStream in = null;
    OutputStream out = null;

    try {
        Paper paper = paperMapper.findPaperByPaperId(paperId);
        String filename = new String(paper.getOriginalName().getBytes("UTF-8"), "ISO8859_1"); // 解决乱码问题
        response.setContentType("application/octet-stream"); // 以流的形式下载文件
        response.setHeader("Content-Disposition", "attachment;filename=" + filename);

        in = new FileInputStream(paperService.getPaperFile(paper));
        out = response.getOutputStream();
        IOUtils.copy(in, out);
    } catch (Exception ex) {
        logger.warn(ex.getMessage());
    } finally {
        IOUtils.closeQuietly(in);
        IOUtils.closeQuietly(out);
    }
}
```


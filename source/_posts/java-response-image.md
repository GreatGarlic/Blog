---
title: HttpServletResponse 返回图片
date: 2016-11-02 21:31:37
tags: [Java, Util]
---
```java
@GetMapping("/image/foo.jpg")
public void enrollRegPhoto(HttpServletResponse response) {
    InputStream in = null;
    OutputStream out = null;

    try {
        response.setContentType("image/png"); // 如果是 jpg 则为 image/jpeg，svg 为 image/svg+xml 等
        in = new FileInputStream("/imageDir/foo.jpg");
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

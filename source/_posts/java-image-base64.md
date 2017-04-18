---
title: 图片和 Base64 字符串互转
date: 2017-04-16 22:26:32
tags: [Java, Util]
---

把图片的二进制数据转换为 Base64 编码的字符串表示，格式为 **data:image/png;base64,iVBORw0KGgoAAAA...**

* `/` 和 `;` 之间的内容为图片的格式
* `,` 后面的内容为图片的二进制数据的Base64 编码的字符串

解析 Base64 编码字符串表示的图片为二进制数据，根据上面的算法反过来解析即可。<!--more-->

## Gradle 依赖

```groovy
compile 'commons-codec:commons-codec:1.10'
compile 'commons-io:commons-io:2.5'
```

## 实现

```java
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.FilenameUtils;

import java.io.File;
import java.io.IOException;

public class ImageBase64Utils {
    public static void main(String[] argv) throws Exception {
        String str = imageToBase64String("/Users/Biao/Pictures/ade.jpg");
        System.out.println(str);

        saveBase64ImageStringToImage("/Users/Biao/Desktop", "y", str);
    }

    /**
     * 把图片转换为 Base64 编码的字符串
     *
     * @param imagePath 图片的路径
     * @return 返回图片的 Base64 编码的字符串
     * @throws IOException
     */
    public static String imageToBase64String(String imagePath) throws IOException {
        // 图片的格式为 data:image/png;base64,iVBORw0KGgoAAAA...
        // 逗号的前面为图片的格式，逗号后们为图片二进制数据的 Base64 编码字符串
        String prefix = String.format("data:image/%s;base64,", FilenameUtils.getExtension(imagePath).toLowerCase());
        byte[] content = FileUtils.readFileToByteArray(new File(imagePath));

        return prefix + Base64.encodeBase64String(content);
    }

    /**
     * 把 Base64 编码的字符串表示的图片保存到传入的目录 directory 下，图片的名字为 baseName，不包含后缀，
     * 图片的后缀从 base64ImageString 中解析得到
     *
     * @param directory 要保存图片的目录
     * @param baseName  图片的名字
     * @param base64ImageString 图片的 Base64 编码的字符串
     * @throws IOException
     */
    public static void saveBase64ImageStringToImage(String directory, String baseName, String base64ImageString) throws IOException {
        // 图片的格式为 data:image/png;base64,iVBORw0KGgoAAAA...
        // 逗号的前面为图片的格式，逗号后们为图片二进制数据的 Base64 编码字符串
        int commaIndex = base64ImageString.indexOf(",");
        String extension = base64ImageString.substring(0, commaIndex).replaceAll("data:image/(.+);base64", "$1");
        byte[] content = Base64.decodeBase64(base64ImageString.substring(commaIndex));

        FileUtils.writeByteArrayToFile(new File(directory, baseName +"."+extension), content);
    }
}
```

## 验证

1. 把图片 A 转为 Base64 编码的字符串

2. 在 HTML 中使用 img 显示 Base64 编码的字符串表示的图片，如果图片显示正常，则说明转换正确:

   ```html
   <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFwAAACOCA...">
   ```

3. 再把上面的字符串解析为图片 B 保存起来

4. 比较上面图片 A 和图片 B 的 MD5，如果一样则说明转化正确
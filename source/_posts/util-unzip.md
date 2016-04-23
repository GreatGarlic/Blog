---
title: Java 解压 zip 文件
date: 2016-04-07 11:01:10
tags: Util
---

使用 Apache `commons-compress` 解压 zip 文件是件很幸福的事，可以解决 zip 包中文件名有中文时跨平台的乱码问题，不管文件是在 Windows 压缩的还是在 Mac，Linux 压缩的，解压后都没有再出现乱码问题了。

<!--more-->

> 例如 Windows 下压缩 `河堤.txt` 为 `河堤.zip`，然后传给 Mac 用户，Mac 用户解压后会发现得到的文件名是乱码，因为 Windows 压缩的时候使用的是系统的编码 GB2312，而 Mac 系统默认的编码是 UTF-8，于是出现了乱码。

## Gradle 依赖
```
dependencies {
    compile 'org.apache.commons:commons-compress:1.11'
    compile 'org.apache.commons:commons-lang3:3.4'
}
```

## 解压程序
```java
import org.apache.commons.compress.archivers.zip.ZipArchiveEntry;
import org.apache.commons.compress.archivers.zip.ZipArchiveInputStream;
import org.apache.commons.compress.utils.IOUtils;
import org.apache.commons.lang3.StringUtils;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class ExtractZip {
    public static final int BUFFER_SIZE = 1024;

    /**
     * 解压 zip 文件
     * @param zipFile zip 压缩文件
     * @param destDir zip 压缩文件解压后保存的目录
     * @return 返回 zip 压缩文件里的文件名的 list
     * @throws Exception
     */
    public static List<String> unZip(File zipFile, String destDir) throws Exception {
        // 如果 destDir 为 null, 空字符串, 或者全是空格, 则解压到压缩文件所在目录
        if(StringUtils.isBlank(destDir)) {
            destDir = zipFile.getParent();
        }

        destDir = destDir.endsWith(File.separator) ? destDir : destDir + File.separator;
        ZipArchiveInputStream is = null;
        List<String> fileNames = new ArrayList<String>();

        try {
            is = new ZipArchiveInputStream(new BufferedInputStream(new FileInputStream(zipFile), BUFFER_SIZE));
            ZipArchiveEntry entry = null;

            while ((entry = is.getNextZipEntry()) != null) {
                fileNames.add(entry.getName());

                if (entry.isDirectory()) {
                    File directory = new File(destDir, entry.getName());
                    directory.mkdirs();
                } else {
                    OutputStream os = null;
                    try {
                        os = new BufferedOutputStream(new FileOutputStream(new File(destDir, entry.getName())), BUFFER_SIZE);
                        IOUtils.copy(is, os);
                    } finally {
                        IOUtils.closeQuietly(os);
                    }
                }
            }
        } catch(Exception e) {
            e.printStackTrace();
            throw e;
        } finally {
            IOUtils.closeQuietly(is);
        }

        return fileNames;
    }

    /**
     * 解压 zip 文件
     * @param zipfile zip 压缩文件的路径
     * @param destDir zip 压缩文件解压后保存的目录
     * @return 返回 zip 压缩文件里的文件名的 list
     * @throws Exception
     */
    public static List<String> unZip(String zipfile, String destDir) throws Exception {
        File zipFile = new File(zipfile);
        return unZip(zipFile, destDir);
    }

    public static void main(String[] args) throws Exception {
        List<String> names = unZip("/Users/Biao/Desktop/Archive.zip", "/Users/Biao/Desktop");
        System.out.println(names);
    }
}
```

## 测试
例如我这测试用的压缩文件是在 Windows 压缩的，解压程序在 Mac 执行的，解压后没有乱码，输出也没有乱码:

> [测试页面.files/, 测试页面.files/image001.jpg, 测试页面.htm]

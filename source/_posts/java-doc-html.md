---
title: Doc 转换为 HTML
date: 2017-03-25 18:43:10
tags: Java
---

上传 doc 文件到服务器上时，如果需要在浏览器中预览，可以把 doc 转为 pdf，然后使用 pdf.js [在线预览 pdf](http://qtdebug.com/pdf-online/)，也可以把 doc 转为 html，直接在浏览器中打开。**POI** 可以很简单的把 doc 转为 html。<!--more-->

## Gradle 依赖

```groovy
compile 'commons-io:commons-io:2.5'
compile 'org.apache.poi:poi-scratchpad:3.16'
```

## 转换程序

```java
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.output.ByteArrayOutputStream;
import org.apache.poi.hwpf.HWPFDocument;
import org.apache.poi.hwpf.converter.PicturesManager;
import org.apache.poi.hwpf.converter.WordToHtmlConverter;
import org.apache.poi.hwpf.usermodel.Picture;
import org.apache.poi.hwpf.usermodel.PictureType;
import org.w3c.dom.Document;

import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.util.List;

/**
 * 把 doc 转换为 html 的工具类
 */
public class DocToHtmlUtils {
    /**
     * 使用 UTF-8 编码，把 doc 文件为 html。
     * html 存储在目录 htmlDir，名字为 index.html，html 的图片存储在目录 htmlDir/img 中。
     *
     * @param docFile  doc 文件
     * @param htmlDir  html 保存的目录
     */
    public static void convertDocToHtml(File docFile, File htmlDir) throws Exception {
        convertDocToHtml(docFile, htmlDir, "index.html", "img", "UTF-8");
    }

    /**
     * 使用指定的编码，把 doc 文件为 html。
     *
     * @param docFile doc 文件
     * @param htmlDir html 保存的目录
     * @param htmlFileName 生成的 html 文件的名字
     * @param imageDirName 生成的 html 的图片存储的目录名，它父目录是 htmlDir
     *                     如果 为 null，则说明图片和 html 文件存放在同一个文件夹下
     * @param encoding 生成 html 文件的编码
     * @throws Exception
     */
    public static void convertDocToHtml(File docFile, File htmlDir, String htmlFileName, String imageDirName, String encoding) throws Exception {
        // 创建 html 和其图片所在文件夹
        File imgDir = htmlDir;

        if (imageDirName == null || imageDirName.trim().equals("")) {
            imageDirName = null;
        } else {
            imageDirName = imageDirName.trim();
            imgDir = new File(htmlDir, imageDirName);
        }

        // 图片文件夹在 htmlDir 内，所以创建它的时候 htmlDir 已经保证先被创建
        FileUtils.forceMkdir(imgDir);

        // 转换 doc 为 html
        HWPFDocument docDocument = new HWPFDocument(new FileInputStream(docFile));
        Document doc = DocumentBuilderFactory.newInstance().newDocumentBuilder().newDocument();
        WordToHtmlConverter docToHtmlConverter = new WordToHtmlConverter(doc);

        final String imgDirName = imageDirName;
        docToHtmlConverter.setPicturesManager(new PicturesManager() {
            public String savePicture(byte[] content,
                                      PictureType pictureType, String suggestedName,
                                      float widthInches, float heightInches ) {
                // HTML 中图片的相对路径
                return imgDirName == null ? suggestedName : imgDirName + File.separator + suggestedName;
            }
        } );

        docToHtmlConverter.processDocument(docDocument);

        // 保存图片
        List<Picture> pictures = docDocument.getPicturesTable().getAllPictures();

        if(pictures != null){
            for (Picture picture : pictures) {
                try {
                    picture.writeImageContent(new FileOutputStream(new File(imgDir, picture.suggestFullFileName())));
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
            }
        }

        // 获取 html 内容
        Document htmlDocument = docToHtmlConverter.getDocument();
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        DOMSource domSource = new DOMSource(htmlDocument);
        StreamResult streamResult = new StreamResult(out);

        TransformerFactory tf = TransformerFactory.newInstance();
        Transformer serializer = tf.newTransformer();
        serializer.setOutputProperty(OutputKeys.ENCODING, encoding);
        serializer.setOutputProperty(OutputKeys.INDENT, "yes");
        serializer.setOutputProperty(OutputKeys.METHOD, "html");
        serializer.transform(domSource, streamResult);
        out.close();

        // 保存 index.html
        FileUtils.writeByteArrayToFile(new File(htmlDir, htmlFileName), out.toByteArray());
    }

    public static void main(String argv[]) throws Exception {
        convertDocToHtml(new File("/Users/Biao/Desktop/doc/x.doc"),
                new File("/Users/Biao/Desktop/doc/x"));
        convertDocToHtml(new File("/Users/Biao/Desktop/doc/x.doc"),
                new File("/Users/Biao/Desktop/doc/y"),
                "goo.html", null, "UTF-8");
    }
}
```


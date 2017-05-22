---
title: Doc 转换为 HTML
date: 2017-03-25 18:43:10
tags: Java
---

上传 doc 文件到服务器上时，如果需要在浏览器中预览，可以把 doc 转为 pdf，然后使用 pdf.js [在线预览 pdf](http://qtdebug.com/pdf-online/)，也可以把 doc 转为 html，直接在浏览器中打开。**POI** 可以很简单的把 doc 转为 html。

Word 里的公式使用的矢量格式 WMF，需要转换为 SVG 后才能在浏览器中查看，下面的代码使用了 wmf2svg 进行了转换。发现图片文件的名字以 **.wmf** 结尾，则给它加上 **.svg**，并且转换为 **SVG** 格式的图片。<!--more-->

## Gradle 依赖

```groovy
compile 'commons-io:commons-io:2.5'
compile 'org.apache.poi:poi-scratchpad:3.16'
compile 'net.arnx:wmf2svg:0.9.8'
```

## 转换程序

```java
import net.arnx.wmf2svg.gdi.svg.SvgGdi;
import net.arnx.wmf2svg.gdi.wmf.WmfParser;
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
import java.io.*;
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
                if (suggestedName.toLowerCase().endsWith(".wmf")) {
                    suggestedName += ".svg"; // 重命名为 xxx.wmf.svg，一会要把 .wmf 图片转换为 svg 的
                }

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
                    String suggestedName = picture.suggestFullFileName();

                    // 如果是 wmf 图片，则转换为 svg
                    if (suggestedName.toLowerCase().endsWith(".wmf")) {
                        FileUtils.writeByteArrayToFile(new File(imgDir, suggestedName + ".svg"), convertWmfToSvg(picture));
                    } else {
                        picture.writeImageContent(new FileOutputStream(new File(imgDir, suggestedName)));
                    }
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

    /**
     * Wmf 转化为 Svg
     *
     * @param wmfPicture
     * @return Svg 内容的 bytes 数组
     * @throws Exception
     */
    public static byte[] convertWmfToSvg(Picture wmfPicture) throws Exception {
        ByteArrayOutputStream wmfContent = new ByteArrayOutputStream();
        wmfPicture.writeImageContent(wmfContent);

        ByteArrayInputStream in = new ByteArrayInputStream(wmfContent.toByteArray());
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        convertWmfToSvg(in, out);

        return out.toByteArray();
    }

    /**
     * 从传入的输入流 in 中读取 wmf 的内容，转换为 svg 格式的内容，保存到 out 中
     *
     * @param in wmf 的输入流
     * @param out svg 的输出流
     * @throws Exception
     */
    public static void convertWmfToSvg(InputStream in, OutputStream out) throws Exception {
        WmfParser parser = new WmfParser();
        SvgGdi gdi = new SvgGdi(false);

        parser.parse(in, gdi);
        Document doc = gdi.getDocument();

        // 保存到 ByteArrayOutputStream
        TransformerFactory factory = TransformerFactory.newInstance();
        Transformer transformer = factory.newTransformer();

        transformer.setOutputProperty(OutputKeys.METHOD, "xml");
        transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
        transformer.setOutputProperty(OutputKeys.INDENT, "yes");
        transformer.setOutputProperty(OutputKeys.DOCTYPE_PUBLIC,"-//W3C//DTD SVG 1.0//EN");
        transformer.setOutputProperty(OutputKeys.DOCTYPE_SYSTEM,"http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd");

        transformer.transform(new DOMSource(doc), new StreamResult(out));
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        transformer.transform(new DOMSource(doc), new StreamResult(buffer));
        out.flush();
    }

    public static void main(String argv[]) throws Exception {
        convertDocToHtml(new File("/Users/Biao/Desktop/2011届高考数学强化复习训练题11.doc"),
                new File("/Users/Biao/Desktop/doc/x"));
//        convertDocToHtml(new File("/Users/Biao/Desktop/doc/x.doc"),
//                new File("/Users/Biao/Desktop/doc/y"),
//                "goo.html", null, "UTF-8");
    }
}
```


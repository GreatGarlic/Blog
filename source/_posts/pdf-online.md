---
title: 在线预览 PDF
date: 2016-12-24 10:30:12
tags: [Mac, Util, FE]
---

下面介绍 2 种在线预览 PDF 的方法，不过都需要 HTML5 的支持，应该问题不大

* 转换 PDF 为 HTML
* 使用 HTML5 的 Canvas 在线绘制 PDF <!--more-->

## 转换 PDF 为 HTML

**pdf2htmlEX** 是一个开源的库，能将 PDF 转换成 HTML，支持 Mac，Linux，Windows，其 github 地址为 [https://github.com/coolwanglu/pdf2htmlEX](https://github.com/coolwanglu/pdf2htmlEX)。

**安装:**

* Mac: brew install pdf2htmlEX
* Linux 和 Windows 请参考 [https://github.com/coolwanglu/pdf2htmlEX/wiki/Building](https://github.com/coolwanglu/pdf2htmlEX/wiki/Building)

**使用:**

* 图片，CSS，JS 等都嵌入到 HTML 中: 

  ```
  pdf2htmlEX test.pdf
  ```

* 图片，CSS，JS 等都被提取出来保存为文件: 

  ```
  pdf2htmlEX --embed cfijo --dest-dir out test.pdf
  ```

* 更多使用说明请参考 [https://github.com/coolwanglu/pdf2htmlEX/wiki/Quick-Start](https://github.com/coolwanglu/pdf2htmlEX/wiki/Quick-Start)

**优点:**

* 开源


* 转换效果真的很完美
* 微信等移动端都支持

**缺点:**

* 浏览器必须支持 HTML5
* 转换出来的文件很大，例如 3M 的 PDF 转换出来大概有 30M

## 使用 HTML5 的 Canvas 在线绘制 PDF

**pdf.js** 是一个主要用于 HTML5 平台上在线阅读 PDF 文档的小插件，基于 JS 技术编写而成，无需任何本地技术支持。

pdf.js 是由 Mozilla Labs 发布的，他们的目标是创建一个通用的，基于标准的网络平台，能够解析和渲染 PDF 文件，并最终发布一个 PDF 阅读器扩展。

pdf.js 的 github 地址为 [https://github.com/mozilla/pdf.js](https://github.com/mozilla/pdf.js)

下面就用最简单的代码来演示 pdf.js 的使用:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Previous/Next example</title>
</head>

<body>

    <h1>'Previous/Next' example</h1>

    <div>
        <button id="prev">Previous</button>
        <button id="next">Next</button>
        &nbsp; &nbsp;
        <span>Page:
            <span id="page_num"></span> /
            <span id="page_count"></span>
        </span>
    </div>

    <div>
        <canvas id="the-canvas" style="border:1px solid black"></canvas>
    </div>

    <!-- for legacy browsers add compatibility.js -->
    <!--<script src="../compatibility.js"></script>-->

    <script src="pdf.js"></script>

    <script id="script">
        //
        // If absolute URL from the remote server is provided, configure the CORS
        // header on that server.
        //
        var url = 'y.pdf';


        //
        // Disable workers to avoid yet another cross-origin issue (workers need
        // the URL of the script to be loaded, and dynamically loading a cross-origin
        // script does not work).
        //
        // PDFJS.disableWorker = true;

        //
        // In cases when the pdf.worker.js is located at the different folder than the
        // pdf.js's one, or the pdf.js is executed via eval(), the workerSrc property
        // shall be specified.
        //
        // PDFJS.workerSrc = '../../build/pdf.worker.js';

        var pdfDoc = null,
            pageNum = 1,
            pageRendering = false,
            pageNumPending = null,
            scale = 1,
            canvas = document.getElementById('the-canvas'),
            ctx = canvas.getContext('2d');

        /**
         * Get page info from document, resize canvas accordingly, and render page.
         * @param num Page number.
         */
        function renderPage(num) {
            pageRendering = true;
            // Using promise to fetch the page
            pdfDoc.getPage(num).then(function(page) {
                var viewport = page.getViewport(scale);
                canvas.height = viewport.height;
                canvas.width = viewport.width;

                // Render PDF page into canvas context
                var renderContext = {
                    canvasContext: ctx,
                    viewport: viewport
                };
                var renderTask = page.render(renderContext);

                // Wait for rendering to finish
                renderTask.promise.then(function() {
                    pageRendering = false;
                    if (pageNumPending !== null) {
                        // New page rendering is pending
                        renderPage(pageNumPending);
                        pageNumPending = null;
                    }
                });
            });

            // Update page counters
            document.getElementById('page_num').textContent = pageNum;
        }

        /**
         * If another page rendering in progress, waits until the rendering is
         * finised. Otherwise, executes rendering immediately.
         */
        function queueRenderPage(num) {
            if (pageRendering) {
                pageNumPending = num;
            } else {
                renderPage(num);
            }
        }

        /**
         * Displays previous page.
         */
        function onPrevPage() {
            if (pageNum <= 1) {
                return;
            }
            pageNum--;
            queueRenderPage(pageNum);
        }
        document.getElementById('prev').addEventListener('click', onPrevPage);

        /**
         * Displays next page.
         */
        function onNextPage() {
            if (pageNum >= pdfDoc.numPages) {
                return;
            }
            pageNum++;
            queueRenderPage(pageNum);
        }
        document.getElementById('next').addEventListener('click', onNextPage);

        /**
         * Asynchronously downloads PDF.
         */
        PDFJS.getDocument(url).then(function(pdfDoc_) {
            pdfDoc = pdfDoc_;
            document.getElementById('page_count').textContent = pdfDoc.numPages;

            // Initial/first page rendering
            renderPage(pageNum);
        });
    </script>
</body>
</html>
```

**上面的例子需要 pdf.js 和 pdf.worker.js 这 2 个文件，去哪里找呢？**

打开例子 [Hello World](http://mozilla.github.io/pdf.js/examples/learning/helloworld.html)，可以从它的源码里找到。

**优点:**

* 开源


* 绘制的效果真的很完美
* 微信等移动端都支持
* 和转换 PDF 为 HTML 的方式比较起来，需要的流量小很多

**缺点:**

* 浏览器必须支持 HTML5
* 如果想一次性显示所有的页，需要修改上面的代码

## 在线预览 PDF 的其他方式

在线预览 PDF 还有很多种其他方法，可以参考 [http://www.open-open.com/news/view/1fc3e18/](http://www.open-open.com/news/view/1fc3e18/)

## 思考

需要在线显示 PPT，也是想到要把 PPT 转换为 HTML 然后在线预览，但是找了好久都没有找到一个免费、并且好用的软件来进行转换，但是把 PPT 转换为 PDF 的免费软件却不少，那么是不是可以考虑先把 PPT 转换为 PDF，然后在通过上面的方法在线预览 PDF 呢？
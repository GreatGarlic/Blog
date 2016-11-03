---
title: Webuploader 上传文件
date: 2016-10-17 22:26:14
tags: FE
---
[SpringMVC Ajax 拖拽上传文件](/spring-upload-file-ajax) 一文介绍了使用 `jQuery File Upload` 拖拽上传文件，也挺简单的，只不过使用的 js 文件比较多，这里介绍使用百度的 `Webuploader` 实现拖拽上传文件，只需要引入 2 个文件:

* webuploader.css
* webuploader.js

有下面一些特性:

* 允许的类型
* 允许的大小
* 创建缩略图
* 图片压缩
* 上传进度
* 拖拽上传
* 自动上传
* 手动上传: `uploader.upload()`
* 更多信息请访问官网 <http://fex.baidu.com/webuploader/>，以及查看 API 文档。

<!--more-->

## 上传文件的页面
`webuploader.html`

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Web Uploader</title>
    <link  href="/lib/webuploader.css" rel="stylesheet" media="screen">
    <script src="/lib/jquery.js" charset="utf-8"></script>
    <script src="/lib/webuploader.js" charset="utf-8"></script>

    <style media="screen">
        body {
            font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
            margin: 50px;
        }

        #drop-area {
            color: gray;
            background: rgb(250,250,250);
            border: dashed 3px lightgray;
            width: 150px;
            height: 150px;
            padding: 20px;
            margin-bottom: 20px;
            line-height: 150px;
            text-align: center;
        }
    </style>
</head>

<body>
    <div id="drop-area">拖拽图片到这里上传</div>
    <div id="uploader-demo">
        <div id="filePicker">点击选择图片</div>
        <div id="fileList" class="uploader-list"></div> <!--用来存放item-->
    </div>


    <script type="text/javascript">
        var uploader = WebUploader.create({
            auto: true,               // 自动上传
            dnd: '#drop-area',        // 拖拽到 #drop-area 进行上传，可选
            swf: '/lib/Uploader.swf', // swf 文件路径
            server: '/webuploader',   // 文件接收服务端 URL
            pick: '#filePicker',      // 选择文件的按钮，内部根据当前运行时创建，可能是 input 元素，也可能是 flash.
            resize: false,            // 不压缩 image, 默认如果是 jpeg，文件上传前会压缩一把再上传！
            accept: { // 只允许上传图片
                title: 'Images',
                extensions: 'gif,jpg,jpeg,bmp,png',
                mimeTypes: 'image/*'
            },
            compress: { // 对上传的图片进行裁剪处理，大于这个分辨率的图片会被压缩到此分辨率
                width: 300,
                height: 300,
                allowMagnify: false,
                crop: false // 是否等比缩放，false 为等比缩放
            }
        });

        // 上传成功
        // response 为服务器返回来的数据
        uploader.onUploadSuccess = function(file, response) {
            console.log(response);
        };

        // 上传成功，例如抛异常
        // response 为服务器返回来的数据
        uploader.onUploadError = function(file, response) {
            console.log(response);
        };

        // 上传进度 [0.0, 1.0]
        // fileQueued 时创建进度条，uploadProgress 更新进度条
        // 可以使用 file.id 来确定是哪个文件的上传进度
        uploader.onUploadProgress = function(file, percentage) {
            console.log(percentage);
            console.log('uploadProgress:' + file.id);
        };

        // 当有文件添加进来的时候
        // 如果是图片，还可以创建缩略图
        uploader.onFileQueued = function(file) {
            console.log('fileQueued:' + file.id);

            var $li = $(
                '<div id="' + file.id + '" class="file-item thumbnail">' +
                '<img>' +
                '<div class="info">' + file.name + '</div>' +
                '</div>'
            );
            var $img = $li.find('img');
            $('#fileList').append($li);

            // 创建缩略图
            // 如果为非图片文件，可以不用调用此方法。
            // src 是 base64 格式的图片
            uploader.makeThumb(file, function(error, src) {
                if (error) {
                    $img.replaceWith('<span>不能预览</span>');
                    return;
                }

                $img.attr('src', src);
            }, 100, 100); // 100 * 100 为缩略图多大小
        };
    </script>
</body>

</html>
```

## 服务器端
没啥好说的，就一普通的 Spring 上传文件，就算前端选择了多个文件，默认时后台也是一次接收到一个，逐个处理。

```java
@Controller
public class WebUploaderController {
    @GetMapping("/webuploader")
    public String webUploaderPage() {
        return "webuploader.html";
    }

    @PostMapping("/webuploader")
    @ResponseBody
    public Result uploadFile(@RequestParam("file") MultipartFile file) throws IOException {
        System.out.println(file.getOriginalFilename());
        file.transferTo(new File("/Users/Biao/Desktop/x/" + file.getOriginalFilename()));
        return new Result(true, "OK", file.getOriginalFilename());
    }
}
```

> 别忘了把 `commons-fileupload` 加入工程，还要在 Spring 的配置文件里创建 `multipartResolver`，否则会报 file 为空指针，怎么都找不到原因
> 
```
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

## 只允许选择上传 jpg 格式的图片
```js
accept: { // 只允许上传图片
    title: 'Images',
    extensions: 'jpg,jpeg',
    // mimeTypes: 'image/*'
    mimeTypes: 'image/jpg,image/jpeg'
}
```

> extensions 指定为 jpg,jpeg，但是如果 mimeTypes 为 image/*，仍然可以在文件选择框中选择其他类型的图片，例如 png，虽然最终是不会上传的，但是这样的行为比较怪异，设置 `mimeTypes: 'image/jpg,image/jpeg'` 后，在文件选择框里就只能选择 jpg, jpeg 的图片了

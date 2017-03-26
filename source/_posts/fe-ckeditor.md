---
title: CKEditor 的使用
date: 2017-03-26 15:13:36
tags: [FE, Java]
---

[CKEditor](http://ckeditor.com) 是一个非常优秀的 Web 服文本编辑器，提供了非常多的功能和丰富的文档

> Constantly leading innovation in the field of rich text editing. Take full control of your content creation process with such unique features as Paste from Word, Advanced Content Filter, widgets, custom HTML formatting and many more.

下面将介绍:

* 集成 CKEditor
* 设置 CKEditor
* 对话框中使用 CKEditor 
* CKEditor 上传图片<!--more-->

## 下载 CKEditor

访问 <http://ckeditor.com>，点击 [Download](http://ckeditor.com/download)，然后下载，标准版就可以了，大多数时候 LGPL 的免费授权协议即可。

## 集成 CKEditor

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>CKEditor 的使用</title>
    <script src="/lib/ckeditor/ckeditor.js"></script>
</head>

<body>
    <textarea id="editor1"></textarea>
    <button onclick="setContent()">Set Content</button>
    <button onclick="getContent()">Get Content</button>

    <script>
        CKEDITOR.replace('editor1'); // 这里的 'editor1' 等于 textarea 的 id 'editor1'

        // 设置 CKEditor 中的内容
        function setContent() {
            // editor1 是上面的 id
            CKEDITOR.instances.editor1.setData('<b>This is for test</b>');
        }

        // 获取 CKEditor 中的内容
        function getContent() {
            var content = CKEDITOR.instances.editor1.getData();
            alert(content);
        }
    </script>
</body>

</html>
```

![](/img/fe/ckeditor-1.png)

## 设置 CKEditor

CKEditor 的设置有 2 种方式，全局设置和局部设置。

上面的例子使用默认的配置使用 CKEditor，很多时候满足不了我们的需求，修改 CKEditor 的配置，去修改 **ckeditor/config.js** 即可，但是这种修改是全局的，所有地方都会受到影响，例如不同页面有可能上传图片的路径应该是不一样的，上传不同类型的文件时上传路径也可能不一样，使用全局的设置就不合适了，这时需要局部的设置 CKEditor，则可以在调用 `CKEDITOR.replace('editor')` 时传入需要的配置参数，例如:

```js
CKEDITOR.replace('editor', {
    language:       'zh-cn', // 语言: 中文，默认是英文
    allowedContent: true,
    removePlugins:  'elementspath', // 编辑器下面不显示元素路径
    resize_enabled: false, // 是否允许拖动改变编辑器的大小
    height:         '300px' // CKEditor 中编辑区的高度，不算工具栏的高度
});
```

## 对话框中使用 CKEditor

某些场景下希望能够在弹出的对话框中进行编辑，例如课程列表中，点击课程后弹出一个对话框，在对话框的编辑器中编辑课程信息，而不是新打开一个页面，这里我们使用 Layer 作为弹出对话框。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>CKEditor 的使用</title>
    <script src="/lib/ckeditor/ckeditor.js"></script>
</head>

<body>
    <!--  -->
    <div id="editor-box" style="display: none; padding: 5px;">
        <textarea id="editor"></textarea>
    </div>
    <button id="show-editor-button">弹出编辑器</button>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>

    <script>
        CKEDITOR.replace('editor');

        $('#show-editor-button').click(function(event) {
            showEditorDialog();
        });

        function showEditorDialog() {
            layer.open({
                type: 1,
                title: '魔法编辑器', // [可选]
                closeBtn: false,
                content: $('#editor-box'), // 对话框中的内容部分
                area: ['700px', '400px'], // 对话框的大小, 可以同时指定高宽 ['700px', '440px']
                shadeClose: false, // 为 true 时点击遮罩关闭
                btn: ['保存', '取消'], // 自定义按钮
                btn1: function() {
                    layer.closeAll(); // 关闭所有弹出窗口
                },
                btn2: function(e) {
                    // return false; // 不关闭弹出窗口
                }
            });
        }
    </script>
</body>

</html>
```
![](/img/fe/ckeditor-2.png)

点击按钮 **弹出编辑器** 看到 CKEditor 显示在了 Layer 中，但是不能在里面编辑，点击全屏，Source 等按钮控制台报错:

> TypeError: undefined is not an object (evaluating 'this.document.getWindow().$.getSelection')

但是，如果这个弹出层使用 Bootstrap 的 Dialog 的话，就没有任何问题，可能是和 Layer 的实现有关吧，不过没关系，Layer 中的问题我们也能搞定: `layer.open()` 的 success() 函数中创建 CKEditor 实例，如果已经存在，则先销毁然后再创建。

> layui.layer 的帮助文档: <http://www.layui.com/doc/modules/layer.html>
>
> success() 是层弹出后的回调方法。

```js
function showEditorDialog() {
    layer.open({
        type: 1,
        title: '魔法编辑器', // [可选]
        closeBtn: false,
        content: $('#editor-box'), // 对话框中的内容部分
        area: ['700px'], // 对话框的大小, 可以同时指定高宽 ['700px', '440px']
        shadeClose: false, // 为 true 时点击遮罩关闭
        btn: ['保存', '取消'], // 自定义按钮
        btn1: function() {
            var content = CKEDITOR.instances.editor.getData();
            alert(content);
            layer.closeAll(); // 关闭所有弹出窗口
        },
        btn2: function(e) {
            // return false; // 不关闭弹出窗口
        },
        success: function() {
            createCkeditor('editor');
        }
    });
}

// 创建 CKEditor 编辑器，如果已经存在，则先销毁
function createCkeditor(name) {
    var editor = CKEDITOR.instances[name];

    if (editor) {
        editor.destroy(true);
    }

    CKEDITOR.replace(name, {
        language: 'zh-cn',
        allowedContent: true,
        removePlugins:  'elementspath',
        resize_enabled: false,
        height: '300px'
    });
}
```

## CKEditor 上传图片

在上面的基础上点击上传图片按钮，可以看到上传图片的层在 Layer 下面，这是因为 Layer 的 z-index 默认是 19891014，而上传图片层的 z-index 是 10000，所以只要修改 Layer 的 z-index 小于 10000 即可，在 open() 函数传参数 `zIndex: 1000` 即可。图片上传的层没问题了，但是发现没有上传图片的选项，只有 图像信息和链接。要显示出上传图片的选项也只是一个配置的问题，只要给 CKEditor 配置 **filebrowserImageUploadUrl** 为上传图片的地址即可。

![](/img/fe/ckeditor-3.png)

```js
function showEditorDialog() {
    layer.open({
        type: 1,
        title: '魔法编辑器', // [可选]
        closeBtn: false,
        content: $('#editor-box'), // 对话框中的内容部分
        area: ['700px'], // 对话框的大小, 可以同时指定高宽 ['700px', '440px']
        shadeClose: false, // 为 true 时点击遮罩关闭
        zIndex: 1000, // *** 重要 ***
        btn: ['保存', '取消'], // 自定义按钮
        btn1: function() {
            var content = CKEDITOR.instances.editor.getData();
            alert(content);
            layer.closeAll(); // 关闭所有弹出窗口
        },
        btn2: function(e) {
            // return false; // 不关闭弹出窗口
        },
        success: function() {
            createCkeditor('editor');
        }
    });
}

// 创建 CKEditor 编辑器，如果已经存在，则先销毁
function createCkeditor(name) {
    var editor = CKEDITOR.instances[name];

    if (editor) {
        editor.destroy(true);
    }

    CKEDITOR.replace(name, {
        language: 'zh-cn',
        allowedContent: true,
        removePlugins: 'elementspath',
        resize_enabled: false,
        height: '300px',
        filebrowserImageUploadUrl: '/ckeditor-upload' // *** 不同页面上传的地址有可能不一样哦 ***
    });
}
```

**服务器端接收图片的程序:**

服务器端接收图片使用参数 **upload**，并且还有一个回调的函数名的参数 **CKEditorFuncNum**，服务器端需要返回的是一段 `<script>`，上传完成后 CKEditor 会调用这一段 `<script>` 例如设置刚才上传的图片到设置框中，或则弹出上传错误信息。

```java
/**
 * 接收 CKEditor 上传的图片
 *
 * @param file 上传的图片
 * @param callback CKEditor 图片上传成功后的回调函数
 * @return 返回一段 <script></script>，CKEditor 会调用这一段 script
 */
@PostMapping("/ckeditor-upload")
@ResponseBody
public String uploadCkeditor(@RequestParam("upload") MultipartFile file, @RequestParam("CKEditorFuncNum") String callback) {
    // [1] 存储上传得到的图片网络地址
    System.out.println(file.getOriginalFilename());

    // [2] 检查图片的格式，如果有错返回错误的 script，如 : <script>alert('图片格式不支持');</script>

    // [3] 上传成功，返回的 script 里带上图片的网络地址
    String imageUrl = "/img/x.png";
    return String.format("<script>window.parent.CKEDITOR.tools.callFunction(%s, '%s')</script>", callback, imageUrl);
}
```

到此为止，CKEditor 的介绍应该能满足百分之八十的需求了，如果想更深度的使用 CKEditor，就需要仔细的阅读帮助文档了 <http://docs.cksource.com/Main_Page>。


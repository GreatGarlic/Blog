---
title: Semantic Ui Validation
date: 2017-03-15 13:24:36
tags: FE
---

Semantic Ui 自带了表单验证功能，使用起来很简单:

* 可以使用预定义的校验函数
* 可以自定义校验函数
* 可以自定义错误提示
* 可以在 **onSuccess** 中禁止表单自动提交，然后使用 Ajax 提交

![](/img/fe/semantic-ui-validation.png)

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <script src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <link href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css" rel="stylesheet">
    <style media="screen">
        body {
            padding: 20px;
        }
    </style>
    <script type="text/javascript">
        $(document).ready(function() {
            // [*] 自定义校验函数
            $.fn.form.settings.rules.passwordStrong = function(value) {
                return value.length > 5;
            };

            $('.ui.form').form({
                fields: {
                    username: {
                        identifier: 'username',  // [*] input 的 name
                        rules: [{
                            type:   'empty',     // [*] 校验规则或函数
                            prompt: '名字不能为空' // [*] 错误提示
                        }]
                    },
                    password: {
                        identifier: 'password',
                        rules: [{
                            type:   'passwordStrong',
                            prompt: '密码强度太弱'
                        }]
                    },
                    color: {
                        identifier: 'color',
                        rules: [{
                            type:   'regExp',
                            value:  /^rgb\((\d{1,3}), (\d{1,3}), (\d{1,3})\)$/i,
                            prompt: '输入合法的颜色，格式为rgb(r, g, b)'
                        }]
                    }
                },
                onSuccess: function(event, fields) {
                    // [*] 表单验证通过后调用 onSuccess 函数
                    // fields 中保存了所有的表单数据，例如 {name: "Alice", color: "rgb(255, 255, 255)"}
                    event.preventDefault(); // [*] 如果需要使用 Ajax 提交时，防止表单自动提交
                    console.log(fields);
                }
            });
        });
    </script>
</head>

<body>
    <form class="ui form segment">
        <div class="two fields">
            <div class="field">
                <label>Username</label>
                <input name="username" placeholder="Enter username" type="text">
            </div>
            <div class="field">
                <label>Password</label>
                <input name="password" placeholder="Enter password" type="password">
            </div>
            <div class="field">
                <label>Color</label>
                <input name="color" placeholder="Enter rgb color" type="text" value="rgb(255, 255, 255)">
            </div>
        </div>
        <div class="ui primary submit button">Submit</div>
        <!-- [*] 错误显示在 error message 中 -->
        <div class="ui error message"></div>
    </form>
</body>

</html>
```

更多内容请参考 [Form Validation](https://semantic-ui.com/behaviors/form.html#adding-custom-rules)
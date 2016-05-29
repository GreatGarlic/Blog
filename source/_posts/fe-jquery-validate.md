---
title: jQuery 表单验证插件 validate
date: 2016-05-27 14:27:41
tags: FE
---

jQuery Validate 插件为表单提供了强大的验证功能，让客户端表单验证变得更简单，同时提供了大量的定制选项，满足应用程序各种需求。该插件捆绑了一套有用的验证方法，包括 URL 和电子邮件验证，同时提供了一个用来编写用户自定义方法的 API，以及使用 Ajax 进行服务器端验证。所有的捆绑方法默认使用英语作为错误信息，且已翻译成其他 37 种语言。
该插件是由 Jörn Zaefferer 编写和维护的，他是 jQuery 团队的一名成员，是 jQuery UI 团队的主要开发人员，是 QUnit 的维护人员。该插件在 2006 年 jQuery 早期的时候就已经开始出现，并一直更新至今。

<!--more-->

## 表单验证实例
需要的依赖有

* jQuery
* Bootstrap
* jQuery Validate

![](/img/fe/jquery-validate.png)

```html
<!DOCTYPE html>
<html>
<head>
    <title>信息填写</title>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <link href="../bootstrap/css/bootstrap.min.css" rel="stylesheet">

    <script src="../jquery.min.js"></script>
    <script src="../bootstrap/js/bootstrap.min.js"></script>
    <script src="jquery.validate.js"></script>

    <style type="text/css">
    body {
        padding-top: 50px;
    }

    .input-group {
        margin-bottom: 10px;
    }

    .button-row {
        margin-top: 10px;
    }

    .alert {
        margin-top: 10px;
    }
    </style>

</head>

<body>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <form id="participant">
                    <div class="input-group">
                        <span class="input-group-addon">姓名</span>
                        <input name="name" type="text" class="form-control">
                    </div>
                    <div class="input-group">
                        <input type="radio" name="gender" value="true" checked> <span>男</span>
                        <input type="radio" name="gender" value="false"> <span>女</span>
                    </div>
                    <div class="input-group">
                        <span class="input-group-addon">电话</span>
                        <input name="telephone" type="text" class="form-control" placeholder="如: 0591-6487256, 15005059587">
                    </div>

                    <div class="input-group">
                        <span class="input-group-addon">邮件</span>
                        <input name="mail" type="text" class="form-control" placeholder="如: xxxx@edu-edu.com.cn">
                    </div>

                    <div class="button-row">
                        <button id="submit-participant" class="btn btn-default btn-block">提交</button>
                    </div>

                    <!-- 显示错误信息的 container -->
                    <div id="error-info" class="alert alert-danger fade in" style="display: none;">
                        <button type="button" class="close" onclick="$('.alert').slideUp()">
                            <span aria-hidden="true">×</span>
                        </button>
                        <div class="errors"></div>
                    </div>
                </form>
            </div>
        </div>
    </div>
</body>

<script type="text/javascript">
    // 正则表达式
    var RegExs = {
        MAIL: /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, // 邮件地址
        TELEPHONE: /^(0[0-9]{2,3}\-)?([2-9][0-9]{6,7})+(\-[0-9]{1,4})?$|(^(13|15|18)\d{9}$)/ // 电话号码
    };

    $(document).ready(function() {
        // 添加自定义的验证函数，例如 notBlank 和 telephone 这 2 个验证规则默认是没有得，可以使用下面的方式自行添加
        // 不能为空字符串
        $.validator.addMethod('notBlank', function(value, element) {
            return $.trim(value);
        });

        // 验证电话号码是否合格
        $.validator.addMethod('telephone', function(value, element) {
            return RegExs.TELEPHONE.test(value);
        });

        // 输入校验:
        // 1. 姓名不能为空或者空字符串
        // 2. 邮件地址不能为空，必须是合法的邮件地址
        // 3. 电话号码不能为空，必须是合法的电话号码，且没有被使用过
        $('#participant').validate({
            // 验证规则
            rules: {
                name: {
                    required: true,
                    notBlank: true
                },
                mail: {
                    required: true,
                    email: true
                },
                telephone: {
                    required: true,
                    telephone: true,
                    // remote: { // Ajax 远程校验，通过校验返回 true, 否则返回 false
                    //     url: Urls.REST_PARTICIPANTS_TELEPHONE_NUMBER_UNUSED,
                    //     type: 'get',
                    //     dataType: 'json',
                    //     data: {
                    //         telephoneNumber: function() {
                    //             return $('#participant input[name="telephone"]').val();
                    //         }
                    //     }
                    // }
                }
            },

            // 错误提示信息，和验证规则一一对应，如果不写，就会使用默认的错误提示信息
            messages: {
                name: {
                    required: '姓名不能为空',
                    notBlank: '姓名不能为空格'
                },
                mail: {
                    required: '邮件不能为空',
                    email:    '请输入合格的邮件地址'
                },
                telephone: {
                    required:  '电话号码不能为空',
                    telephone: '请输入合格的电话号码',
                    remote:    '电话号码已经使用过'
                }
            },

            // 自定义错误显示的位置，默认是显示在 <input> 的后面，很多时候不符合需求
            // 错误提示信息的 <label> 包裹在 <li> 里然后添加到 <div class="erros"> 中
            errorLabelContainer: $('#error-info div.errors'),
            wrapper: 'li',
            showErrors: function(errorMap, errorList) {
                // 有错误的时候才显示
                if (this.numberOfInvalids() > 0) {
                    $('#error-info').show();
                } else {
                    $('#error-info').hide();
                }

                this.defaultShowErrors();
            },
            submitHandler: function(form) {
                // 点击 '提交' 按钮，验证通过，会调用这个函数，就可以在这里进行 form 表单的提交，例如 Ajax 的方式
                // 也可以不实现这个函数，表单提交时使用表单的默认方式提交
            }
        });
    });

</script>
</html>
```

## 参考
* [jQuery 下载](http://jquery.com)
* [jQuery validate 下载](http://plugins.jquery.com/validation/)
* [Bootstrap 下载](http://getbootstrap.com)
* [jQuery validate 官网](https://jqueryvalidation.org/documentation/)
* [jQuery validate 教程](http://www.runoob.com/jquery/jquery-plugin-validate.html)

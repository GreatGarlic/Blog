---
title: Semantic Ui 进度条
date: 2017-03-16 22:57:09
tags: [FE, Semantic-Ui]
---

Semantic Ui 使用 **progress** 来创建进度条，可以使用 **data-percent** 或者 **data-value + data-total** 两种方式计算及显示进度:

![](/img/fe/semantic-ui-progress.png)

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <script src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <link  href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css" rel="stylesheet">
    <style media="screen">
        body {
            padding: 20px;
        }
    </style>

</head>

<body>
    <!-- [1] 使用百分比的进度条 -->
    <div class="ui olive progress" data-percent="67" id="progress1">
        <div class="bar">
            <div class="progress"></div> <!-- 如果不想显示百分比，去掉这一行 -->
        </div>
    </div>
    <button class="ui secondary button decrease">Decrease</button>

    <div class="ui divider"></div>

    <!-- [2] 自定义完成度的进度条 -->
    <div class="ui purple progress" data-value="0" data-total="20" id="progress2">
        <div class="bar">
            <div class="progress"></div>
        </div>
        <div class="label">Adding Photos</div>
    </div>
    <button class="ui secondary button increase">Increase</button>

    <script type="text/javascript">
        $(document).ready(function() {
            // [3] 初始化进度条
            $('#progress1').progress();
            $('#progress2').progress({
                text: {active: '{value} of {total} done'}
            });

            // [4] 点击修改 progress1 的进度为 22%
            $('.decrease.button').click(function(event) {
                $('#progress1').progress({percent: 22});
            });

            // [5] 点就修改 progress2 完成了 20 个中的 7 个，百分比会自动计算
            $('.increase.button').click(function(event) {
                $('#progress2').progress({
                    value: 7,
                    text: {
                        active: '{value} of {total} done'
                    }
                });
            });
        });
    </script>
</body>

</html>
```

更多详细信息请参考 <https://semantic-ui.com/modules/progress.html#/usage>
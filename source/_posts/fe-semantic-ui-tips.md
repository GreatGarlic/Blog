---
title: Semantic Ui Tips
date: 2017-05-04 14:44:43
tags: [FE, Semantic-Ui]
---

## JS and CSS

一下代码都是基于 jQuery、Semantic Ui、Layer、Vue 来写的:

```html
<link rel="stylesheet" href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css">
<script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
<script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
<script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
<script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>
```

<!--more-->

## Dropdown

```html
<div class="ui floating labeled icon dropdown button">
    <input type="hidden">
    <i class="world icon"></i>
    <span class="text">选择地区</span>
    <div class="menu">
        <div class="item" data-value="1">北京</div>
        <div class="item" data-value="2">上海</div>
        <div class="item" data-value="3">天津</div>
        <div class="item" data-value="4">湖北</div>
    </div>
</div>
```

```js
// 初始化方式一：
$('.dropdown').dropdown();

// 初始化方式二：Dropdown，当选项变化时触发 onChange() 函数
$('.dropdown').dropdown({onChange: function(value, text, $choice) {
    layer.msg(value + " : " + text);
}});

// 选择 data-value="2" 的项为当前选项
$('.dropdown').dropdown('set selected', 2);

// 获取当前选项的 value
$('.dropdown').dropdown('get value');

// 获取当前选项的 text
$('.dropdown').dropdown('get text');
```

## Card

![](/img/fe/semantic-ui-card-2.png)

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css">
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>

    <style media="screen">
        body {
            padding: 20px;
        }

      	/* [[5]] (可选)校正 dimmer 的圆角*/
        .ui.card .dimmer {
            cursor: default;
            border-top-left-radius: .28571429rem;
            border-top-right-radius: .28571429rem;
        }
    </style>
</head>

<body>
    <div class="ui card">
        <!-- [[1]] 图片 -->
        <div class="dimmable image">
            <div class="ui dimmer">
                <div class="content">
                    <div class="center">焚我残躯，熊熊圣火，生亦何欢，死亦何苦？为善除恶，唯光明故。喜乐悲愁，皆归尘土。怜我世人，忧患实多！</div>
                </div>
            </div>
            <img src="http://omqpd0pt4.bkt.clouddn.com/steve.jpg">
        </div>
        <!-- [[2]] 主体内容 -->
        <div class="content">
            <div class="header">Matt Giampietro</div>
            <div class="meta"><a>Friends</a></div>
            <div class="description">Matthew is an interior designer living in New York. </div>
        </div>
        <!-- [[3]] 额外内容例如按钮等 -->
        <div class="extra content">
            <span><i class="tags icon"></i></span>
            <span><i class="user icon"></i> 75 Friends</span>
            <span class="right floated">Joined in 2013</span>
        </div>
    </div>

    <script>
        // [[4]] (可选)鼠标放到 dimmable 上显示半透明的 dimmer
        $('.dimmable').dimmer({on: 'hover'});
    </script>
</body>

</html>
```

## Popup

* 使用 **data-tooltip**，不需要 JS，Popup 的位置不会自动变化:

  ```html
  <body>
      <div class="ui icon button" data-tooltip="Add users to your feed">
          <i class="add icon"></i>
      </div>

      <div class="ui icon button" data-tooltip="Add users to your feed" data-position="bottom left" data-inverted="">
          <i class="minus icon"></i>
      </div>
  </body>
  ```

* 使用 **data-content**，需要执行 popup() 函数，Popup 的位置能够自适应:

  ```html
  <body>
      <span class="with-popup" data-content="知识点标签"><i class="tags icon"></i></span>

      <script>
          // 2 种方式都可以
          $('.with-popup').popup();
          $('.with-popup').popup({ position: 'bottom left' });
      </script>
  </body>
  ```

* 自定义 Popup，需要执行 popup() 函数，Popup 的内容和触发者不在一起:

  ![](/img/fe/semantic-ui-popup.png)

  ```html
  <body>
      <!-- [[1]] Popup trigger -->
      <div class="ui button" id="show-popup-button">Show custom popup</div>
      <span id="show-popup-span"><i class="tags icon"></i> Google</span>

      <!-- [[2]] Popup 的内容，可以是很丰富的 HTML -->
      <div class="ui custom popup transition hidden">
          I'm not on the same level as the <div class="ui grey button">Button</div>,
          but i can still be found.
      </div>

      <script>
          // [[3]] 初始化 Popup
          $('#show-popup-button, #show-popup-span').popup({
              position: 'bottom left',
              popup: $('.custom.popup'),
              on: 'hover' // hover, click
          });
      </script>
  </body>
  ```

  ​


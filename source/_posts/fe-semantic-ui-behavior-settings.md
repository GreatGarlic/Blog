---
title: Semantic Ui 的 Behavior 和 Settings
date: 2017-03-25 09:14:19
tags: [FE, Semantic-Ui]
---

学习 Semantic Ui 的时候，不少组件的文档里都有 behavior 和 settings，例如 Dimmer, Form Validation, Transition, Tab, Dropdown等，相信不少同学对 behavior 和 settings 应该怎么用摸不着头脑。

Behavior 就是行为，和函数是一回事，写个函数不就好了？但是 Semantic Ui 里偏偏不这样，而是传入 behavior 的名字，然后内部去调用。

Settings(设置) 更是不知道什么时候用啊，既然是设置，那么逻辑上就应该在 behavior 被调用之前先行设置好，这样在 behavior 被调用的时候才会生效，如果在 behavior 被调用后才使用 settings，那么就没有意义了。

从参数上就可以区分是 Settings 还是 Behavior:

* Settings: 参数是一个 JSON 对象

* Behavior: 第一个参数是一个字符串，behavior 的名字，后面是其他参数，例如

  ```js
  $('.dropdown').dropdown('set selected', 2);
  ```

下面就以 card 中使用 dimmer 为例，介绍 behavior 和 settings 的使用:

![](/img/fe/semantic-ui-card.png) <!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <link href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css" rel="stylesheet">
    <style media="screen">
        body {
            font-family: 'Arial';
            padding: 50px;
        }
        .dimmer {
            font-size: 12px;
        }
    </style>
</head>

<body>
    <div class="ui two column grid">
        <div class="ui column">
            <div class="ui card" id="one">
                <div class="ui image dimmerable">
                    <img src="http://tupian.enterdesk.com/2016/gha/02/1902/02.jpg.200.150.jpg">
                    <div class="ui dimmer">
                        <div class="content">
                            <div class="center">天之道，损有馀而补不足，是故虚胜实，不足胜有馀。其意博，其理奥，其趣深。天地之像分，阴阳之侯烈，变化之由表，死生之兆章。</div>
                        </div>
                    </div>
                </div>
                <div class="extra content">九阴真经</div>
            </div>
        </div>

        <div class="ui column">
            <div class="ui card" id="two">
                <div class="ui image dimmerable">
                    <img src="http://tupian.enterdesk.com/2015/gha/12/1202/02.jpg.200.150.jpg">
                    <div class="ui dimmer">
                        <div class="content">
                            <div class="center">运气导行、移宫使劲的法门，悟性高者7年可成，差一点的14年才能练成。</div>
                        </div>
                    </div>
                </div>
                <div class="extra content">乾坤大挪移</div>
            </div>
        </div>

        <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
        <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
        <script>
            // 鼠标进入 dimmerable 上时显示 dimmer
            $(document).on('mouseenter', '.ui.dimmerable', function() {
                $('.ui.dimmer', this).dimmer('show');
            });

            // 鼠标离开 dimmerable 上时隐藏 dimmer
            $(document).on('mouseleave', '.ui.dimmerable', function() {
                $('.ui.dimmer', this).dimmer('hide');
            });

        </script>
</body>

</html>
```

页面显示时 dimmer 是隐藏起来的，当鼠标鼠标进入 dimmerable 上时响应 mouseenter 事件显示 dimmer: `$('.ui.dimmer', this).dimmer('show')`，其中 **show** 就是 dimmer 的 behavior（show 和 hide 是很多组件默认的 behavior），望文生义，使用 behavior `dimmer('hide')` 就是隐藏 dimmer，这就是 behavior 的使用。Dimmer 的 behavior 通过 **dimmer()** 函数来调用，Dropdown 的 behavior 通过 **dropdown()** 函数来调用，具体的参考帮助文档。

不过上面显示和隐藏 dimmer 的动画效果有些单调，阅读 dimmer 的帮助文档，可以看到 Settings 中有 transition（动画效果），不过 dimmer 文档的 Definition, Examples, Usage, Settings 中找遍了都没有看到介绍 transition 怎么使用呢。如果想用 drop 的动画效果显示 dimmer 应该怎么办呢？

```js
$('.ui.dimmer', this).dimmer({transition: 'drop'}).dimmer('show');
```

上面的代码就是 settings 的使用，大家有没有注意到，settings 是一个 JSON 对象，作为 `dimmer()` 函数的参数，虽然 behavior 使用的时候也是使用 `dimmer()` 函数，但是参数是一个字符串，虽然函数是同一个，但通过参数的类型来判断使用的是 settings 还是 behavior。

**Settings 有 2 种使用方式，局部的设置和全局的设置:**

* **局部设置:** 上面的例子中就是使用了局部的 settings，只会对选择器匹配的元素生效，局部设置会覆盖全局设置。如下的代码分别对 **#one** 和 **#two** 中的 dimmer 使用不同的 settings:

  ```js
  $(document).on('mouseenter', '#one .ui.dimmerable', function() {
      $('.ui.dimmer', this).dimmer({
          transition: 'drop',
          duration: 800,
          opacity: 0.2
      }).dimmer('show');
  });

  $(document).on('mouseenter', '#two .ui.dimmerable', function() {
      $('.ui.dimmer', this).dimmer({
          transition: 'scale',
          duration: 200
      }).dimmer('show');
  });
  ```

* **全局设置:** 使用 `$.fn.dimmer.settings` 进行 dimmer 的全局设置:

  ```js
  $.fn.dimmer.settings.transition = 'vertical flip';
  $.fn.dimmer.settings.opacity = 0.3;
  $.fn.dimmer.settings.duration = 800;

  // 鼠标进入 dimmerable 上时显示 dimmer
  $(document).on('mouseenter', '.ui.dimmerable', function() {
      $('.ui.dimmer', this).dimmer('show');
  });
  ```

到此相信大家对 behavior 和 settings 的使用已经掌握的差不多了，至于每个组件都有哪些 behavior 和 settings，就需要大家花时间去阅读 Semantic Ui 的帮助文档了。
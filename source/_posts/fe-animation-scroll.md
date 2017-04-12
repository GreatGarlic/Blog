---
title: 滚动插件 Animation Scroll
date: 2017-04-12 13:04:58
tags: FE
---

AnimationScroll 是一个 jQuery 的滚动插件，支持很多缓冲动画效果

> **AnimateScroll** is a jQuery plugin which enables you to **scroll to any part of the page in style**by just calling the `animatescroll()` function with the `Id` or `Classname` of the element where you want to scroll to.
>
> **Basic usage:** `$('body').animatescroll();`

![](/img/fe/animate-scroll.png)

可到 <https://plugins.jquery.com/animatescroll/> 下载，主页为 <http://plugins.compzets.com/animatescroll/>。

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <style media="screen">
        .element-demo {
            overflow: auto;

            height: 150px;
            margin: 0.5em 0;
            padding: 1em;

            color: grey;
            border: 0.3em solid #545454;
            border-radius: 0.5em;
            background: none repeat scroll 0 0 #141414;
            box-shadow: 1px 1px 0.5em black inset;
            text-shadow: 0 -0.1em 0.2em black;

            font-family: Monaco, Menlo, monospace;
        }
    </style>
</head>

<body>
    <div class="element-demo">
        <p>
            <button>Click here</button> to scroll to the last paragraph within this "div" element
        </p>
        <p>
            This "div" element has a class-name "element-demo" which is the value passed for "element" option while calling the plugin.
        </p>
        <p>
            Compzets.com is India's first open source software and freeware publishing site, Download and Upload Open source software and Freeware relevant to the Paid ones for PC,Mac and Linux.
        </p>
        <p>
            It also makes its own Cloud Applications for making tasks easy. Recently it has launched a new Plugin Showcase too.
        </p>
        <p id="last-paragraph">
            This website is your source of unprecedented access to all kinds of pc,mac or linux software (Open Source or Freeware only) with detailed coverage of tech infos along with multiple screen shots and moreover you can not only download your favorite gadgets
            but you can also UPLOAD your own software to reach thousand of audience. Stay connected to all the latest happenings in the gadget world,with regular updates on new software and announcements with the help of our RSS Feed,just at a few clicks!
        </p>
        <p>
            The word "Compzets" does not have a literal meaning,it is just derived from the word Gadget which is related to Electronic devices where as Compzets is related to Computer software which are nothing but gadgets for computer.
        </p>
        <p>
            Thanks to <a href="https://plus.google.com/114685591029748634833" target="_blank">Ronan DMP</a> for asking this feature!
        </p>
    </div>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="animatescroll.js"></script>
    <script>
        $('button').click(function(event) {
            // #last-paragraph 是 .element-demo 的子元素
            $('#last-paragraph').animatescroll({
                element: '.element-demo',
                padding: 10,
                scrollSpeed: 300
            });
        });
    </script>
</body>

</html>
```


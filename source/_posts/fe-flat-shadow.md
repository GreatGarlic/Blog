---
title: FlatShadow
date: 2016-12-02 22:41:45
tags: FE
---

A small jQuery plugin that will automatically cast a shadow creating depth for your flat UI elements Created by [Pete R.](http://www.thepetedesign.com/), Founder of [BucketListly](http://www.bucketlistly.com/), 主页请访问 [https://github.com/peachananr/flat-shadow](https://github.com/peachananr/flat-shadow)

![](/img/fe/flat-shadow.png)

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>
    <script src="jquery.flatshadow.min.js" charset="utf-8"></script>

    <style media="screen">
        body {
            font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
        }

        .flat-icon {
            display: inline-block;
            width: 50px;
            height: 50px;
            list-style: none;
            text-align: center;
            border-radius: 50px;
            line-height: 50px;
        }
    </style>
</head>

<body>
    <ul>
        <li class="flat-icon">FLAT</li>
        <li class="flat-icon">UI</li>
    </ul>
    <script type="text/javascript">
        $(".flat-icon").flatshadow({
            color: "#AAAABBDD", // Background color of elements inside. (Color will be random if left unassigned)
            angle: "SE", // Shadows direction. Available options: N, NE, E, SE, S, SW, W and NW. (Angle will be random if left unassigned)
            fade: true, // Gradient shadow effect
            // boxShadow: "#d7cfb9" // Color of the Container's shadow
        });
    </script>
</body>

</html>
```


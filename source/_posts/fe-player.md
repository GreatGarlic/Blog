---
title: HTML5 播放器
date: 2017-08-19 18:52:07
tags: FE
---

[Video.js](http://www2.videojs.com) 是一个简洁、漂亮的 HTML5 播放器，支持字幕，还可支持 Flash(不支持 HTML5 时自动切换到 Flash)，使用很简单，也能自定义插件:

> Video.js is a JavaScript and CSS library that makes it easier to work with and build on HTML5 video. This is also known as an HTML5 Video Player. Video.js provides a common controls skin built in HTML/CSS, fixes cross-browser inconsistencies, adds additional features like fullscreen and subtitles, manages the fallback to Flash or other playback technologies when HTML5 video isn't supported, and also provides a consistent JavaScript API for interacting with the video.

![](/img/fe/video.js.png)

<!--more-->

下面是 Video.js 最简单的例子([源码](/download/video-js.7z)):

```html
<!DOCTYPE html>

<html>

<head>
    <title>Video.js | HTML5 Video Player</title>
    <link href="video-js.min.css" rel="stylesheet" type="text/css">
    <script src="video.min.js"></script>
    <script>
        videojs.options.flash.swf = "video-js.swf"; // 不使用 Flash 的可以删除
    </script>
</head>

<body>
    <video id="x-video" class="video-js vjs-default-skin vjs-big-play-centered"
        controls autoplay preload="metadata" poster="poster.png"
        width="640" height="360" data-setup="">
        <source src="x.mp4" type="video/mp4"/>
    </video>
</body>

</html>
```

> 提示: 
>
> * 给 **video** 一个 **id** 属性很有必要，这样 video 会被包裹在一个 id 为 video 的 id 的 div 里，方便以后操作，例如要修改 video 的大小: `$('#video-id').width(320).height(180)` 
>
>
> * 不能删除 **data-setup** 这个属性，删除了 Video.js 就不会生效了
> * 为了效果更好，video 的宽高设置为视频的宽高，一般 **16:9** 的比较多

更详细的文档请阅读 [GET STARTED](https://github.com/videojs/video.js/blob/stable/docs/guides/setup.md) 和 [DOCS](https://github.com/videojs/video.js/blob/stable/docs/index.md)


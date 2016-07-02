---
title: 自定义 HTML5 播放器
date: 2016-06-30 16:49:59
tags: FE
---

HTML5 默认提供的播放器如果满足不了我们的需求，可以对其进行自定义，例如隐藏默认的按钮，然后定义播放的进度条、进度条上某个时刻的提示、按钮等，常用的 API 有:

* 属性: `paused`
* 属性: `muted`
* 属性: `duration`
* 属性: `currentTime`
* 函数: `play()`
* 函数: `pause()`
* 更多 API 请参考 <http://www.w3.org/2010/05/video/mediaevents.html>

<!--more-->

## 自定义 HTML5 播放器
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>自定义 HTML5 播放器</title>

    <style media="screen">
        #player-container {
            width: 600px;
        }

        #slider {
            width: 100%;
            display: block;
        }

        #player {
            width: 100%;
        }

        #play-or-pause-button {
            width: 60px;
        }
    </style>
</head>

<body>
    <div id="player-container">
        <!-- 删除 controls 不显示播放器默认的工具栏 -->
        <video id="player" src="http://china-bigdatauniversity.oss-cn-qingdao.aliyuncs.com/BD801EN%2FL02V01_GettingStarted.mp4" autoplay></video>

        <div id="player-control-bar">
            <input id="slider" type="range" name="name" value="0" min="0" max="1000" step="1">
            <button id="play-or-pause-button">暂停</button>
        </div>
    </div>

    <script src="jquery.min.js" charset="utf-8"></script>
    <script type="text/javascript">
        $(document).ready(function() {
            var player = $('#player').get(0); // 播放器，必须使用 Dom Element 才可以

            // 播放或者暂停
            $('#play-or-pause-button').click(function() {
                if (player.paused) {
                    player.play();
                    $(this).text('暂停');
                } else {
                    player.pause();
                    $(this).text('播放');
                }
            });

            // 拖动 slider 时改变播放进度
            $('#slider').change(function() {
                var progress = $(this).val() / 1000;
                player.currentTime = player.duration * progress; // player.duration 视频的总时长
            });

            // 播放进度的回调
            $('#player').on('timeupdate', function() {
                var progress = player.currentTime / player.duration * 1000;
                $('#slider').val(progress);
            });
        });
    </script>
</body>

</html>
```

## 自定义 YouTube 播放器
YouTube 的播放器已经提供了很多高级功能，我们也对其进行自定义: <http://tutorialzine.com/2015/08/how-to-control-youtubes-video-player-with-javascript/>

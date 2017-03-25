---
title: 首字母放大缩进
date: 2017-03-23 18:58:57
tags: FE
---

![](/img/fe/first-letter.png)

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <style media="screen">
        body {
            font-family: 'Arial'
        }
        div {
            text-indent: 2em;
        }
        div:first-letter {
            font-size: 3em;
            color: gray;
        }
    </style>
</head>

<body>
    <div>焚我残躯,熊熊烈火.生亦何欢,死亦何苦.为善除恶,惟光明故.喜乐悲愁,皆归尘土.怜我世人,忧患实多.怜我世人,忧患实多.</div>
    <div>焚我残躯,熊熊烈火.生亦何欢,死亦何苦.为善除恶,惟光明故.喜乐悲愁,皆归尘土.怜我世人,忧患实多.怜我世人,忧患实多.</div>
</body>

</html>
```


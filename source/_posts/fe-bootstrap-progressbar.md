---
title: Bootstrap Progress Bar
date: 2016-05-26 17:09:02
tags: FE
---

```html
<div class="progress">
    <div class="progress-bar" style="width: 2%;"><span>2% 是不是显示不完整</span></div>
</div>
```

上面的代码显示出的进度条中由于进度只有 2%，只显示出了 `2%`，`是不是显示不完整` 没有显示出来
![](/img/fe/bootstrap-progressbar-1.png)

<!--more-->

---

要想把 `2% 是不是显示不完整` 全部显示出来，使用下面的 CSS 即可

```css
.progress {
    position:relative;
}

.progress span {
    position: absolute;
    left: 0;
    width: 100%;
    z-index: 2;
    color: black;
}
```

![](/img/fe/bootstrap-progressbar-2.png)

---
title: jQuery 中 fadeIn() 和 slideDown() 同时执行
date: 2016-05-20 20:54:12
tags: FE
---

jQuery 没有直接提供 fadeIn() 和 slideDown() 同时执行的函数，但是可以像下面这样实现:

```js
$elem.stop(true, true).fadeIn({ duration: 300, queue: false }).css('display', 'none').slideDown(300);
```

> The answer is that once either effects activate, it takes the inline css property "display=none" off of the element. These hide effects require the display property to be set to "none". So, just rearrange the order of methods and add a css-modifier in the chain between fade and slide.

fadeOut() 和 slideUp() 同时执行的代码如下:

```js
$elem.stop(true, true).fadeOut({ duration: 300, queue: false }).slideUp(300);
```

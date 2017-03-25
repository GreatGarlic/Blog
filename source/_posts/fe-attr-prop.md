---
title: jQuery 的 attr 和 prop
date: 2017-03-23 15:04:00
tags: FE
---
jQuery 中可以使用 attr 和 prop 获取的属性，它们的区别是什么呢？

> The segregation of [`attr()`](http://api.jquery.com/attr) and [`prop()`](http://api.jquery.com/prop) should help alleviate some of the confusion between HTML attributes and DOM properties. `$.fn.prop()` grabs the specified DOM property, while `$.fn.attr()` grabs the specified HTML attribute.

HTML attributes 在网页的代码中可以看到，DOM properties 是内存数据，在 HTML 代码中看不到。**attr** 操作的是 HTML attributes, **prop** 操作的是 DOM properties，它们可以是重叠的，例如 img 的 src 既可以使用 attr 访问，也可以使用 prop 访问，但是 checkbox 的 checked 应该使用 prop 访问，attr 访问会有奇怪的问题。大多数时候 attr 和 prop 的效果都一样，但是访问 boolean 的属性时切记要使用 prop。<!--more-->

* 读写 HTML 元素本身就带有的固有属性，也即是 DOM properties，**尤其是 boolean 的属性**，使用prop方法，例如

  * checked
  * selected
  * disabled
  * readOnly

* 读写自定义的 HTML 属性使用 attr，例如创建一个新的属性 data-avatar，当然例如 img 的 src 等使用 attr 读写是没有问题的

  > 自定义属性使用 prop 来写的话，不会生成到 HTML 的代码中，但是数据能够读取出来，不过因为看不到，所以很多时候就不方便调试。

The easiest way to see the difference between **attr** and **prop** is the following example:

```html
<input blah="hello">
```

* `$('input').attr('blah')`: returns **hello** as expected. No suprises here.
* `$('input').prop('blah')`: returns **undefined** -- because it's trying to do `[HTMLInputElement].blah` -- and no such property on that DOM object exists. It only exists in the scope as an attribute of that element i.e. `[HTMLInputElement].getAttribute('blah')`


```js
$('input').attr('blah', 'apple');
$('input').prop('blah', 'pear');
```

* `$('input').attr('blah')`: returns **apple**, not **pear** as this was set last on that element. Because the property was changed on the input attribute, not the DOM input element itself -- they basically almost work independently of each other.
* `$('input').prop('blah')`: returns **pear**

> A property can contain things of different types. An attribute can only contain strings.
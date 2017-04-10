---
title: RequireJS 加载非 AMD 的 JS
date: 2017-04-06 19:25:26
tags: FE
---

RequireJS 加载的 JS 要求是 AMD 规范的，但是非 AMD 规范的 JS 文件也能够加载，下面就以 Util.js 中定义了类 Rect，Circle 和普通函数 greeting() 为例，演示 RequireJS 对于非 AMD 规范的 JS 的配置，加载以及使用。

可以看到，使用 RequireJS 加载的 JS 中的类和函数，与使用 `<script src="/js/Util.js">` 加载时的使用方式没有区别，如果要以 AMD 的方式使用非 AMD 的 JS，可以参考 <http://www.bubuko.com/infodetail-671521.html><!--more-->

## RequireJS 配置

```js
// 文件名: require-config.js
require.config({
    paths: {
        Util: '/js/Util'
    }
});
```

> 不需要 shim 和 exports，如果 Util 依赖 jQuery 的话，就需要 shim 了，例如
>
```js
require.config({
    paths: {
        jquery: '/lib/jquery',
        Util: '/js/Util'
    },
    shim: {
        Util: {
            deps: ['jquery']
        }
    }
});
```


## 网页

```html
<!-- 文件名: b.html -->
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
</head>

<body>
    <script src="/lib/require.js"></script>
    <script src="/js/require-config.js"></script>
    <script>
        require(['Util'], function() { // 不需要 module 的名字
            // 使用 RequireJS 加载的 Util.js 中的类和函数，与使用 <script src="/js/Util.js"> 加载时的使用方式没有区别
            var rect = new Rect(10, 20, 50, 50); // 创建类的对象
            console.log(rect.pos()); // 成员函数调用

            var circle = new Circle(0, 0, 20);
            console.log(circle.area()); // 成员函数调用
            Circle.foo(); // 类函数调用

            greeting(); // 普通函数调用
        });
    </script>
</body>

</html>
```

## 普通 JS 文件

```js
// 文件名: Util.js
// [1] 类 Rect
function Rect(x, y, width, height) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
}

Rect.prototype.pos = function() {
    return {
        x: this.x,
        y: this.y
    };
};

// [2] 类 Circle
function Circle(x, y, radius) {
    this.x = x;
    this.y = y;
    this.radius = radius;
}

// 成员函数
Circle.prototype.area = function() {
    return 3.1415 * this.radius * this.radius;
};

// 类的函数
Circle.foo = function() {
    console.log('Circle::foo()');
};

// [3] 普通函数
function greeting() {
    console.log('Greeting...');
}
```

## 测试

访问 <http://localhost/b.html>，输出

```
{x: 10, y: 20}
1256.6000000000001
Circle::foo()
Greeting... 
```

​
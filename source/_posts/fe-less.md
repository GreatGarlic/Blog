---
title: 使用 LESS 代替 CSS
date: 2017-03-17 14:23:09
tags: FE
---

使用 Less 最直接的好处是可以使用层级的风格写样式，当然它还可以定义变量、函数等，比 CSS 好管理，而且语法和 CSS 差别不大，这里介绍 HTML 里使用 Less 代替 CSS:

1. 书写 Less 文件

2. HTML 中引入 Less 文件

   ```html
   <link href="style.less" rel="stylesheet/less" type="text/css"/>
   ```

3. HTML 中引入 less.js，它的作用是把上面引入的 Less 文件解析为 CSS，放到 HTML 的 head 中

   ```html
   <script src="https://cdn.staticfile.org/less.js/2.7.1/less.min.js"></script>
   ```

<!--more-->

## 案例

```css
// 文件名: style.less

// 定义变量
@borderWidth: 2px;
@borderColor: #BBB;

.card {
    padding: 10px;
    border: @borderWidth solid @borderColor;
    border-radius: 4px;

    // 层级的风格写 CSS
    .header {
        font-size: 30px;
        font-weight: bold;
        font-family: Tahoma;
        text-align: center;

        .sub.header {
            font-size: 15px;
            font-weight: normal;
            color: gray;
        }
    }

    .content {
        background: #EEE;
        padding: 4px;
        color: black;
    }
}

```

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Less</title>

    <!-- [1] 引入 less 写的样式文件 -->
    <link href="style.less" rel="stylesheet/less" type="text/css"/>
    <!-- [2] 引入 less.js 把上面的 style.less 转换成 css 样式-->
    <script src="https://cdn.staticfile.org/less.js/2.7.1/less.min.js"></script>
</head>

<body>
    <!-- Less 中的样式生效 -->
    <div class="card">
        <div class="header">
            Input
            <div class="sub header">
                Will be used in form
            </div>
        </div>
        <div class="content">An input is a field used to elicit a response from a user</div>
    </div>

    <!-- 不受 Less 中的样式影响 -->
    <div class="header">
        Will be used in form
    </div>
</body>

</html>
```

效果为:

![](/img/fe/less-1.png)

如果查看网页的元素，会发现在 head 里生成了 **style**:

![](/img/fe/less-2.png)

> 上面的 Less 解析为 CSS 花了大概 14 毫秒，性能应该不是问题，而且是在浏览器端解析的，对服务器没有任何影响。
>
> If you're using a 3rd party CMS, have limited control of server side, and don't want to worry about syncing issues with the produced CSS, it's an option. As the stack overflow discussions point out, Less caches the translated CSS in newer browsers. But our site also requires JavaScript to function anyway, so if JS is turned off, they can't really use the site anyway. Don't get me wrong, we're still looking for a preprocessing method, **but so far performance hasn't really been an issue**.

## Less 语法简介

### 变量

Less 的**变量名**使用 **@** 符号开始：

```css
@mainColor: #0982c1;
@siteWidth: 1024px;
@borderStyle: dotted;
 
body {
    color: @mainColor;
    border: 1px @borderStyle @mainColor;
    max-width: @siteWidth;
}
```

### Mixins

**Mixins** 有点像是**函数或者是宏**，当你某段 CSS 经常需要在多个元素中使用时，你可以为这些共用的 CSS 定义一个 Mixin，然后你只需要在需要引用这些 CSS 地方调用该 Mixin 即可:

```css
/* LESS mixin error with (optional) argument @borderWidth which defaults to 2px if not specified */
.error(@borderWidth: 2px) {
    border: @borderWidth solid #F00;
    color: #F00;
}
 
.generic-error {
    padding: 20px;
    margin: 4px;
    .error(); /* Applies styles from mixin error */
}
.login-error {
    left: 12px;
    position: absolute;
    top: 20px;
    .error(5px); /* Applies styles from mixin error with argument @borderWidth equal to 5px */
}
```

### 注释

有两种注释风格，但是有细微区别:

* `// comments` 的注释不会出现在 style 中
* `/* comments*/` 的注释会出现在 style 中

### 嵌套

如果我们需要在CSS中相同的 parent 引用多个元素，这将是非常乏味的，你需要一遍又一遍地写 parent。例如：

```css
section {
    margin: 10px;
}

section nav {
    height: 25px;
}

section nav a {
    color: #0982C1;
}

section nav a:hover {
    text-decoration: underline;
}
```

而如果使用 Less，就可以少些很多单词，而且父子节点关系一目了然，下面是 Less 的嵌套语法：

```css
section {
    margin: 10px;
    nav {
        height: 25px;
        a {
            color: #0982C1;
            &:hover {
                text-decoration: underline;
            }
        }
    }
}
```

## 参考资料

* [Less 快速入门](http://less.bootcss.com)
* [为您详细比较三个 CSS 预处理器（框架）：Sass、LESS 和 Stylus](http://www.oschina.net/question/12_44255?sort=default&p=6)


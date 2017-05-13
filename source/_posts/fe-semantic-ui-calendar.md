---
title: Semantic Ui 日期插件
date: 2017-05-13 10:59:23
tags: [FE, Semantic-Ui]
---

Semantic Ui 没有提供日期控件，在网上找到了一个基于它的实现，能够选择年、日期、时间、日期+时间，日期范围等，功能很齐全。

可以使用 npm 安装: `npm install --save semantic-ui-calendar`，也可以下载 [semantic-ui-calendar.7z](/download/semantic-ui-calendar.7z) 直接使用，其外观如下:

![](/img/fe/semantic-ui-calendar.png)

<!--more-->

## 使用步骤

1. 引入 SemanticUi 和 SemanticUi Calendar 的 css 和 js:

   ```html
   <link href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css" rel="stylesheet">
   <link href="calendar.css" rel="stylesheet" >

   <script src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
   <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
   <script src="calendar.js" charset="utf-8"></script>
   ```

2. 定义 calendar 的 input:

   ```html
   <div class="ui calendar" id="example1">
       <div class="ui input left icon">
           <i class="calendar icon"></i>
           <input type="text" placeholder="Date" value="2017-06-01">
       </div>
   </div>
   ```

3. 初始化 calendar:

   ```js
   $('#example1').calendar({type: 'date'});
   ```

## 自定义

* 自定义日期格式，默认格式如 **June 1, 2017** 这样的，很多时候我们可能需要 **2017-01-06** 这样通用的格式，则需要在初始化的时候提供 formatter 函数:

  ```js
  $('#example1').calendar({
      type: 'date',
      formatter: { // 自定义日期的格式
          date: function(date, settings) {
              if (!date) return '';

              var year  = date.getFullYear();
              var month = date.getMonth() + 1;
              var day   = date.getDate();

              month = month < 10 ? '0'+month : month;
              day   = day   < 10 ? '0'+day   : day;

              return year + '-' + month + '-' + day;
          }
      }
  });
  ```
  > 当然也能修改源码的 formatter 实现自定义格式，这样就不用在用到的地方都单独的提供 formatter 函数。

* 自定义语言，默认只提供了英文，如果需要使用中文的话，需要修改源码里的 text 对象:

  ```js
  text: {
      days: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],
      months: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
      monthsShort: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
      today: 'Today',
      now: 'Now',
      am: 'AM',
      pm: 'PM'
  }
  ```


## 参考资料

See <http://codepen.io/SaadRegal/pen/ZOABQr> for example usage.
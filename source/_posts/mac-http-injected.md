---
title: 一次 HTTP 被运营商劫持的血泪史
date: 2018-04-19 20:19:46
tags: Mac
---

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">

<head>
    <script src="/js/jquery.js" charset="utf-8"></script>
    <script src="/js/paper.js" charset="utf-8"></script>
</head>

<body>
    <script>
        $(document).ready(function() {
            __Exam_PaperInit();
        });
    </script>
</body>

</html>
```

很简单的页面，jquery.js 和 paper.js 加载完，然后调用 `__Exam_PaperInit()` 进行初始化。在办公室、家里都没出现过问题，但是在学校的机房里访问这个页面，时不时的出错，提示如 `__Exam_PaperInit()` 不存在，这么简单直接的逻辑，咋会出错呢，想不明白，猜测例如是不是机房的环境网络设置有问题等，但花了很久仍然找不到原因，更要命的是，系统过几天就有几千人要用来考试了，问题解决不了的话，可以想象影响会有多大。<!--more-->

多番查找资料，分析出错时的网络加载情况，发现了些奇怪的地方 (开始的时候都视而不见，因为看到了也不知道是啥)：

![](/img/mac/http-injected.png)

发现如上图 paper.js 加载了 2 次，第一次加载的大小明显不对，第二次加载的时候大小对了，但还加上了个时间戳，我们的程序里没有加这个时间戳啊，哪里来的？还发现个奇怪的请求 `/a`，我们的网页里没有发过这个请求，点开它一看，具体的 URL 是 <http://124.232.160.178/v1/a>，很奇怪的 IP，到网上一搜索，发现是运营商的广告拦截，这下拨云见日、豁然开朗了：

1. 加载 jquery.js

2. 加载 paper.js，但是它被运营商拦截，返回的内容逻辑如

   ```
   request('http://xxx.com/js/paper.js')
   request('http://124.232.160.178/v1/a')
   ```

3. 发起异步加载 paper.js 和 /a

4. jquery.js 和 ~~paper.js~~ 因为都加载完了，所以接下来执行 `__Exam_PaperInit()` 初始化，结果出错，因为这时的 paper.js 不是真正的 paper.js，而是运营商劫持后返回来的，所以里面是不存在 `__Exam_PaperInit()` 的

5. 异步加载 paper.js 完成，可以看到下面的 `paper.js?_t1524016700292=0i` 请求，它的 Initiator 是 paper.js (被劫持)，这也就明白了为啥 paper.js 有 2 个请求

6. 当真正的 paper.js 加载完的时候已经晚了，因为初始化函数早就执行完了

问题终于弄清楚了，搞不定运营商，投诉没用啊，只能在代码和服务器上想办法了，有 2 个办法：

* 网站使用 HTTPS，这样就能防止被运营商劫持了，不用修改一点代码，但是升级到 HTTPS 需要申请证书、配置 Nginx 或者服务器，也不是一下会就能搞好的，不懂的人更是困难重重，不过这是最好的方案

* 循环检查、确保需要的对象 `jQuery` 和 `__Exam_PaperInit` 都存在时才进行初始化:

  ```html
  <!DOCTYPE html>
  <html lang="en" dir="ltr">

  <head>
      <script src="/js/jquery.js" charset="utf-8"></script>
      <script src="/js/paper.js" charset="utf-8"></script>
  </head>

  <body>
      <script>
          var intervalId = setInterval(function() {
              if (typeof jQuery == 'undefined') { return; }
              if (typeof __Exam_PaperInit == 'undefined') { return; }
              clearInterval(intervalId);

              __Exam_PaperInit();
          }, 100);
      </script>
  </body>

  </html>
  ```

  ​


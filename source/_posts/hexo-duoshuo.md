---
title: Hexo 多说评论
date: 2016-04-03 17:49:03
tags: Hexo
---

在博客系统里增加评论功能，如果自己做的话，需要准备服务器端(PHP 和 Java 等实现)和数据库用于评论的提交、读取和保存。如果只是简单的使用评论功能，对评论的数据分析、安全性等要求不高，可以使用第三方提供的评论系统，例如 `多说`，集成起来也很简单，只要在页面的 HTML 代码里加入一小段代码即可。

<!-- more -->

## 申请多说二级域名
访问 <http://duoshuo.com/create-site>，例如申请的多说二级域名为 <http://xtuer.duoshuo.com>

* 则多说的 shortname 为 `xtuer`
* 管理网址: <http://xtuer.duoshuo.com/admin>

## 添加多说的短名字
在根目录下 `_config.yml` 中增加 `duoshuo_shortname: xtuer`

## 添加评论代码
如果使用的是默认的 landscape 主题只需要修改 `themes\landscape\layout\_partial\article.ejs` 中的

```
<% if (!index && post.comments && config.disqus_shortname){ %>
<section id="comments">
    <div id="disqus_thread">
        <noscript>Please enable JavaScript to view the <a href="//disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
    </div>
</section>
<% } %>
```

为
    
```
<% if (!index && post.comments && config.duoshuo_shortname){ %>
<section id="comments">
    <!-- 多说评论框 start -->
    <div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
    <!-- 多说评论框 end -->
    
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
        var duoshuoQuery = {short_name:'<%= config.duoshuo_shortname %>'};
        (function() {
            var ds = document.createElement('script');
            ds.type = 'text/javascript';ds.async = true;
            ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
            ds.charset = 'UTF-8';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
        })();
    </script>
    <!-- 多说公共JS代码 end -->
</section>
<% } %>
```
    
> 如果是其他主题，需要修改主题 `\layout\_partial\comment.ejs`，根据具体情况判断

> 注意 `data-thread-key` 一定不要改变，`data-thread-key` 相当于是识别码，如果文章的 `data-thread-key` 改变了的话，那么恭喜你，评论全部没有了 (每篇文章可以使用一个唯一的 `UUID` 作为 `data-thread-key`)

## 参考
* [在 Hexo 中加入多说评论](http://www.lichanglin.cn/在hexo中加入多说评论/)

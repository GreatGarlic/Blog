---
title: 小程序中显示 HTML
date: 2017-02-15 14:09:58
tags: FE
---
小程序本身没有提供显示 HTML 的功能，可以使用第三方插件例如 [WxParse](https://github.com/icindy/wxParse) 来显示 HTML。
<!--more-->

## 使用方法
1. 复制 wxParse 目录到小程序的项目中

    ```
    └── wxParse
        ├── emojis        (可选)
        ├── html2json.js  (必须存在)
        ├── htmlparser.js (必须存在)
        ├── showdown.js   (必须存在)
        ├── wxDiscode.js  (必须存在)
        ├── wxParse.js    (必须存在)
        ├── wxParse.wxml  (必须存在)
        └── wxParse.wxss  (必须存在)
    ```
    emojis 有 300 多 K，小程序最大为 1M，所以不需要 emoji 支持时可以删除掉
2. 引入 wxParse.wxss  
    在需要使用 WxParse 的页面的 wxss 文件中引入 wxParse.wxss，也可以在 app.wxss 中全局引入
    
    ```css
    @import "/wxParse/wxParse.wxss";
    ```
3. 引入 wxParse.js  
    在 js 文件中引入 wxParse.js
    
    ```js
    var WxParse = require('../../wxParse/wxParse.js');
    ```
4. 数据绑定  
    例如在 onLoad() 函数中绑定 HTML 数据
    
    ```js
    Page({
        data:{},
        onLoad:function(options){
            // article 为 HTML 内容
            var article = '<div>这里是 HTML 代码，可以有内联的 CSS 样式，图片等，<a> 标签不能点击</div>';
          
            /**
             * 1. bindName 绑定的数据名(必填)，在模版中引用
             * 2. type 可以为 html 或者 md(必填)
             * 3. data 为传入的具体数据(必填)
             * 4. target 为 Page 对象, 一般为 this(必填)
             * 5. imagePadding 为当图片自适应是左右的单一 padding(默认为 0, 可选)
             *
             * WxParse.wxParse(bindName, type, data, target, imagePadding)
             */
            WxParse.wxParse('article', 'html', article, this, 0);
        }
    })
    ```
5. 使用模版显示 HTML  
    在 wxml 文件中使用模版显示 HTML
    
    ```html
    <import src="../../wxParse/wxParse.wxml"/>
    
    <!-- 这里 data 中的 article 为 bindName -->
    <template is="wxParse" data="{{wxParseData:article.nodes}}"/>
    ```

    > 注意:
    > 
    > * CSS 样式必须内联的嵌入在 HTML 中，不能放在单独的文件里，通过 link 引用
    > * 模版的名字是固定的 **wxParse**

下面附上具体的例子
## html.wxss
```css
@import "/wxParse/wxParse.wxss"; /* [0] */

.title-view {
    display: flex; 
    justify-content: center; 
    font-weight: bold; 
    font-size: 40rpx; 
    padding: 10rpx;
    border-bottom: 1rpx solid #CCC; 
}
```

## html.js
```js
var WxParse = require('../../wxParse/wxParse.js'); // [1]

Page({
    data: {},
    onLoad: function(options) {
        // [2]
        var article = '<div>这里是 HTML 代码<img src="http://img06.tooopen.com/images/20160818/tooopen_sy_175866434296.jpg"><img src="http://img02.tooopen.com/images/20150928/tooopen_sy_143912755726.jpg"></div>';
        WxParse.wxParse('article', 'html', article, this, 0);
    }
});
```

## html.wxml
```html
<!-- [3] -->
<import src="../../wxParse/wxParse.wxml" />

<view>
    <view class="title-view">小程序中渲染 HTML</view>
    <view style="padding: 10rpx;">
        <!-- [4] -->
        <template is="wxParse" data="{{wxParseData:article.nodes}}"/>
    </view>
</view>
```

显示效果:

![](/img/fe/wx-html.png)

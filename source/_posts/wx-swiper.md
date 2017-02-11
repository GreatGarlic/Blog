---
title: 小程序轮播
date: 2017-02-08 10:39:13
tags: FE
---
微信小程序使用 **Swiper** 实现图片的轮播，**Swiper** 的结构如下:

```html
<swiper indicator-dots="{{indicatorDots}}" 
        circular="{{circular}}"
        autoplay="{{autoPlay}}" 
        interval="{{interval}}" 
        duration="{{duration}}">
    <swiper-item><image src=""/></swiper-item>
    ...
    <swiper-item><image src=""/></swiper-item>
</swiper>
```
**swiper-item** 的子标签可以是 image，或者 navigator 中放图片实现点击跳转等。

![](/img/fe/wx-swiper.png)
<!--more-->

```html
<!-- 文件名: swiper.wxml -->

<swiper indicator-dots="{{indicatorDots}}" 
        circular="{{circular}}"
        autoplay="{{autoPlay}}" 
        interval="{{interval}}" 
        duration="{{duration}}">
    <swiper-item wx:for="{{imgUrls}}" wx:for-item="url">
        <image src="{{url}}" class="slide-image" mode="widthFix" style="width: 100%;"/>
    </swiper-item>
</swiper>

<button bindtap="changeIndicatorDots">indicator-dots</button>
<button bindtap="changeAutoPlay">autoplay</button>

<view class="slider-box">
    Interval <slider bindchange="intervalChange" show-value min="500" max="10000"/>
</view>
<view class="slider-box">
    Duration <slider bindchange="durationChange" show-value min="1000" max="10000"/>
</view>
<view>
    <image style="width: 100%;" mode="widthFix" src="http://img06.tooopen.com/images/20160818/tooopen_sy_175833047715.jpg"/>
</view>
```
> * image 的 mode 设置为 widthFx，这样使得图片的高度通过 CSS 制定，高度自动的通过等比缩放计算得到。
> * 如果 duration 的值比 interval 的小，会有奇怪的问题，估计设计的时候 interval 的值包含了动画的 duration 时间和动画结束后的静止时间

```css
/* 文件名: swiper.wxss */

slider {
    flex: 1;
}

.slider-box {
    display: flex; 
    flex-direction: row; 
    align-items: center;
}
```

```js
// 文件名: swiper.wxss

Page({
    data: {
        imgUrls: [
            'http://img06.tooopen.com/images/20160818/tooopen_sy_175866434296.jpg',
            'http://img06.tooopen.com/images/20160818/tooopen_sy_175833047715.jpg',
            'http://img02.tooopen.com/images/20150928/tooopen_sy_143912755726.jpg'
        ],
        indicatorDots: true,
        autoPlay: true,
        circular: true,
        interval: 2000,
        duration: 1000
    },
    changeIndicatorDots: function(e) {
        this.setData({
            indicatorDots: !this.data.indicatorDots
        });
    },
    changeAutoPlay: function(e) {
        this.setData({
            autoPlay: !this.data.autoPlay
        });
    },
    intervalChange: function(e) {
        this.setData({
            interval: e.detail.value
        });
    },
    durationChange: function(e) {
        this.setData({
            duration: e.detail.value
        });
    }
});
```

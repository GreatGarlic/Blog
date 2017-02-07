---
title: 小程序布局
date: 2017-02-07 10:18:45
tags: FE
---
微信小程序布局使用的是 Flex，只是某些地方稍微有些变种，下面演示小程序中主要使用到的 Flex 布局。

![](/img/fe/wx-flex.png)

<!--more-->

```xml
<!--文件名: flex.wxml-->

<view class="container">
    <view class="flex-section">
        <text>justify-content: flex-start</text>
        <view class="flex-items" style="justify-content: flex-start;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>

    <view class="flex-section">
        <text>justify-content: flex-end</text>
        <view class="flex-items" style="justify-content: flex-end;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>

    <view class="flex-section">
        <text>justify-content: center</text>
        <view class="flex-items" style="justify-content: center;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>

    <view class="flex-section">
        <text>justify-content: space-between</text>
        <view class="flex-items" style="justify-content: space-between;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>

    <view class="flex-section">
        <text>justify-content: space-around</text>
        <view class="flex-items" style="justify-content: space-around;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>

    <view class="flex-section">
        <text>align-items: center 垂直居中</text>
        <view class="flex-items" style="justify-content: space-around; height: 200rpx; align-items: center;">
            <view class="flex-item bc_red"></view>
            <view class="flex-item bc_green"></view>
            <view class="flex-item bc_blue"></view>
        </view>
    </view>
</view>
```

```css
/* 文件名: flex.wxss */

.container {
    background-color: #EEE;
}

/** 
 * 盒子模型中，width 默认指的是 content 的宽度，如果想 width 表示包含边框在内的宽度，
 * 则需要设置 box-sizing: border-box; 这样 border，padding，content 等加起来的
 * 宽度为 width
 */
.flex-section {
    width: 100%;
    margin-bottom: 10rpx;
    background-color: white;
    box-sizing: border-box; /* 很重要，因为设置了 padding，否则 width: 100% 则超出了 parent 的 width */
    padding: 10rpx;
}

.flex-section:last-child {
    margin-bottom: 0;
}

.flex-items {
    display: flex; 
    flex-direction: row;
    margin-top: 10rpx;
}

.flex-item {
    width: 100rpx;
    height: 100rpx;
}

.bc_red {
    background-color: #E68C23;
}

.bc_green {
    background-color: #FCEB62;
}

.bc_blue {
    background-color: #1DAF94;
}
```


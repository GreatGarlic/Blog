---
title: 去掉 png 图片的 iCCP 警告
date: 2018-04-22 23:07:55
tags: Qt
---

Qt 中使用 png 图片有时候会给出警告 `libpng warning: iCCP: known incorrect sRGB profile`:

> Libpng-1.6 is more stringent about checking ICC profiles than previous versions. You can ignore the warning. To get rid of it, remove the iCCP chunk from the PNG image.
>
> Some applications treat warnings as errors; if you are using such an application you do have to remove the chunk. 

解决办法：

1. 安装 [ImageMagick](http://www.imagemagick.org/script/download.php) (Mac: `brew install ImageMagick`)
2. 到图片文件夹，执行命令 `mogrify *.png` 去掉此文件夹下 png 图片的 iCCP 警告

要想找出有 iCCP 问题的 png 图片，可以使用工具 [pngcrush](https://pmt.sourceforge.io/pngcrush/):

1. 安装 [pngcrush](https://pmt.sourceforge.io/pngcrush/) (Mac: `brew install pngcrush`)
2. 到图片文件夹，执行命令 `pngcrush -n -q *.png` 找出有 iCCP 警告的图片

更多细节请参考 [libpng warning: iCCP: known incorrect sRGB profile](https://stackoverflow.com/questions/22745076/libpng-warning-iccp-known-incorrect-srgb-profile)。
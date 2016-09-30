---
title: 升级 Xcode8 后 Qt 出错
date: 2016-09-29 20:00:20
tags: Qt
---

错误提示:

> Project ERROR: Xcode not set up properly. You may need to confirm the license agreement by running /usr/bin/xcodebuild.

解决办法:

1. 打开 `Qt_install_folder/5.7/clang_64/mkspecs/features/mac/default_pre.prf`
2. 修改 

    ```
    isEmpty($$list($$system("/usr/bin/xcrun -find xcrun 2>/dev/null")))
    ```

    为

    ```
    isEmpty($$list($$system("/usr/bin/xcrun -find xcodebuild 2>/dev/null")))
    ```
    
具体请看 <http://stackoverflow.com/questions/33728905/qt-creator-project-error-xcode-not-set-up-properly-you-may-need-to-confirm-t>


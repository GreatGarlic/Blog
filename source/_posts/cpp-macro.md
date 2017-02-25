---
title: C++ 查看预处理后的源文件
date: 2017-02-24 11:54:54
tags: Qt
---
`gcc -E filename.cpp` 会生成 filename.cpp 的预处理文件，这样就能看到宏展开后的代码，用于理解和调试宏非常有帮助。

![](/img/qt/macro-1.png)
![](/img/qt/macro-2.png)

---
title: 字节序 Endian
date: 2017-04-12 19:48:30
tags: [Java, Qt]
---

Endian 一词来源于[乔纳森·斯威夫特](http://baike.baidu.com/view/177903.htm)的小说[格列佛游记](http://baike.baidu.com/view/79427.htm)。小说中，小人国为水煮蛋该从大的一端（Big-End）剥开还是小的一端（Little-End）剥开而争论，争论的双方分别被称为 Big-Endians 和 Little-Endians。

1980 年，Danny Cohen 在其著名的论文 "*On Holy Wars and a Plea for Peace*" 中为平息一场关于[字节](http://baike.baidu.com/view/60408.htm)该以什么样的顺序传送的争论而引用了该词。

**Endian** 翻译为**“字节序”**，又称**端序**，**尾序**。在[计算机科学](http://baike.baidu.com/view/92404.htm)领域中，**字节序**是指存放多字节数据的字节（byte）的顺序，典型的情况是整数在[内存](http://baike.baidu.com/view/1082.htm)中的存放方式和[网络传输](http://baike.baidu.com/view/1542295.htm)的传输顺序。Endianness 有时候也可以用指**位序**（bit）。

一般而言，字节序指示了一个 UCS-2 [字符](http://baike.baidu.com/view/263416.htm)的哪个字节存储在低地址。如果 LSByte 在 MSByte 的前面，即 LSB 为低地址，则该字节序是**小端序**；反之则是**大端序**。在[网络编程](http://baike.baidu.com/view/1317473.htm)中，字节序是一个必须被考虑的因素，因为不同的[处理器](http://baike.baidu.com/view/50152.htm)体系可能采用不同的字节序。在多平台的代码编程中，字节序可能会导致难以察觉的 [bug](http://baike.baidu.com/view/1743.htm)。

**BIG Endian**：最低位地址存放高位字节，可称高位优先，内存从最低地址开始按顺序存放（高数位数字先写）。最高位字节放最前面。

**LITTLE Endian**：最低位地址存放低位字节，可称低位优先，内存从最低地址开始按顺序存放（低数位数字先写）。最低位字节放最前面。

## Big Endian 解释

最低位地址存放高位字节，可称高位优先，内存从最低地址开始按顺序存放（高数位数字先写）。最高位字节放最前面。

例如“汉”字的 Unicode 编码是 6C49。如果将 6C 写在前面，就是 big endian，如果将 49 写在前面，就是 little endian。


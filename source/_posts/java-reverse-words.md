---
title: 逆转字符串
date: 2017-04-12 19:43:19
tags: Java
---

逆转字符串，但是其中的单词仍然是正序的:

1. 先逆转整个字符串
2. 再逆转逆转后的字符数组中组成单词的字符<!--more-->

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(reverseWords("How are you!"));
        System.out.println(reverseWords("do justice to a dinner"));
        System.out.println(reverseWords(""));
        System.out.println(reverseWords("A"));
        System.out.println(reverseWords("A Biao"));
        System.out.println(reverseWords("!A Biao C."));
    }

    public static String reverseWords(String str) {
        int start = 1;
        int end = 0;
        char[] chs = str.toCharArray();

        // [1] 先逆转整个字符串
        reverseCharactersInRange(chs, 0, chs.length - 1);

        // [2] 再逆转逆转后的字符数组中组成单词的字符
        for (int i = 0; i < chs.length; ++i) {
            if (Character.isLetter(chs[i])) {
                // 找到组成单词的字符的起始和结束位置
                if (start > end) {
                    start = end = i;
                } else {
                    ++end;
                }
            } else {
                if (start < end) {
                    reverseCharactersInRange(chs, start, end);
                }
                start = chs.length;
            }
        }

        if (start < end) {
            reverseCharactersInRange(chs, start, end);
        }

        return new String(chs);
    }

    public static void reverseCharactersInRange(char[] chs, int start, int end) {
        int times = (end - start + 1) / 2;

        for (int i = 0; i < times; ++i) {
            char temp = chs[start + i];
            chs[start + i] = chs[end - i];
            chs[end - i] = temp;
        }
    }
}
```


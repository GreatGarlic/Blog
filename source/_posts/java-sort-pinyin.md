---
title: Java 按照拼音排序
date: 2018-01-10 15:25:52
tags: [Java, Util]
---

Java 中按照拼音序对字符串排序，只需要使用 CHINA 的 Collator 即可

```java
import java.text.Collator;
import java.util.LinkedList;
import java.util.List;
import java.util.Locale;

public class Test {
    public static void main(String[] args) {
        List<String> tokens = new LinkedList<>();
        tokens.add("黄"); // h
        tokens.add("慌"); // h
        tokens.add("晃"); // h
        tokens.add("欢"); // h
        tokens.add("中"); // z
        tokens.add("这"); // z
        tokens.add("国"); // g
        tokens.add("古"); // g
        tokens.add("安"); // a

        Collator collator = Collator.getInstance(Locale.CHINA);
        tokens.sort((a, b) -> collator.compare(a, b));

        System.out.println(tokens);
    }
}
```


---
title: How to change default Java version
date: 2016-04-16 10:36:12
tags: Java
---

#### 1. First run `/usr/libexec/java_home -V` which will output something like the following

>```
Matching Java Virtual Machines (2):
    1.8.0_77, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home
    1.7.0_45, x86_64:	"Java SE 7"	/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home
```

<!--more-->

#### 2. Pick the version you want to be the default

>```
export JAVA_HOME=`/usr/libexec/java_home -v 1.7.0_45`
```

#### 3. Now when you run `java -version` you will see

>```
java version "1.7.0_45"
Java(TM) SE Runtime Environment (build 1.7.0_45-b18)
Java HotSpot(TM) 64-Bit Server VM (build 24.45-b08, mixed mode)
```

#### 4. Just add the above `export JAVA_HOME …` line to your shell’s init file

> Such as `<user-home>/.bash_profile`

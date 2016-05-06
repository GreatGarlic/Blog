---
title: Java 使用 FTP
date: 2016-05-06 09:11:17
tags: [Java, Util]
---

Java 使用 FTP 等用 [`Apache Commons Net`](http://commons.apache.org/proper/commons-net/index.html) 就可以了:

> Apache Commons Net™ library implements the client side of many basic Internet protocols. The purpose of the library is to provide fundamental protocol access, not higher-level abstractions. Therefore, some of the design violates object-oriented design principles. Our philosophy is to make the global functionality of a protocol accessible (e.g., TFTP send file and receive file) when possible, but also provide access to the fundamental protocols where applicable so that the programmer may construct his own custom implementations (e.g, the TFTP packet classes and the TFTP packet send and receive methods are exposed).

<!--more-->

Supported protocols include:

* FTP/FTPS
* FTP over HTTP (experimental)
* NNTP
* SMTP(S)
* POP3(S)
* IMAP(S)
* Telnet
* TFTP
* Finger
* Whois
* rexec/rcmd/rlogin
* Time (rdate) and Daytime
* Echo
* Discard
* NTP/SNTP

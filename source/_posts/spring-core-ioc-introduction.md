---
title: Spring IoC Introduction
date: 2017-04-01 15:14:53
tags: SpringCore
---

## IoC 是什么？

控制反转（Inversion of Control，英文缩写为 IoC）是一个重要的面向对象编程的法则来削减计算机程序的耦合问题，也是轻量级的 Spring 框架的核心。 控制反转一般分为两种类型，`依赖注入`（Dependency Injection，简称 DI）和`依赖查找`（Dependency Lookup）。依赖注入应用比较广泛。

应用控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体(`Spring IoC Container`)将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。所以，控制反转是，关于一个对象如何获取他所依赖的对象的引用，这个责任的反转。

对象的生成不再是在代码里用 new 创建的，而是在 XML 里定义对象之间的依赖关系，然后由 Spring 来生成对象。

## IoC 最大的好处是什么？

* 因为把对象生成放在了 XML 里定义，所以当我们需要换一个实现子类将会变成很简单（一般这样的对象都是实现于某种接口的），只要修改 XML 就可以了，这样我们甚至可以实现对象的热插拔（有点像 USB 接口和 SCSI 硬盘了）。

## IoC 最大的缺点是什么？

* 生成一个对象的步骤变复杂了（事实上操作上还是挺简单的），对于不习惯这种方式的人，会觉得有些别扭和不直观。
* 对象生成因为是使用反射编程，在效率上有些损耗。但相对于 IoC 提高的维护性和灵活性来说，这点损耗是微不足道的，除非某对象的生成对效率要求特别高。
* 缺少 IDE 重构操作的支持，如果在 Eclipse 要对类改名，那么你还需要去XML文件里手工去改了，这似乎是所有 XML 方式的缺憾所在。
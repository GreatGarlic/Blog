---
title: Velocity 语法
date: 2017-07-01 10:47:01
tags: SpringWeb
---

Velocity 比较接近脚本语言，例如 JS

```
#if ($foo < 10)
    ...
#elseif ($foo == 10)
    ...
#elseif ($foo == 12)
    ...
#else
    ...
#end
```

比较一下 Freemarker

```
<#if foo < 10> 
    ... 
<#elseif foo == 10> 
    ... 
<#elseif foo == 12> 
    ... 
<#else>
    ...
</#if>
```

<!--more-->

## 注释

* 单行注释: `##`

  ```
  ## comment
  ```

* 多行注释: `#* *#`

  ```
  #* line1
  line2
  line3
  ...
  lineN *#
  ```

## 变量

* **$** 用来标识一个变量(或理解为对象)，如

  ```
  $msg
  $msg.author
  $msg.toUpperCase()

  <!-- Map 也可以使用 . 访问: users 是一个 Map, alice 是 key -->
  $users.alice
  ```

  > 可以级联调用对象属性，调用对象的函数

* **{}** 用来明确标识 Velocity 变量

  比如在页面中，页面中有一个 `$someonename`，此时，Velocity 将把someonename 作为变量名，若我们程序是想在 someone 这个变量的后面紧接着显示 name 字符，则上面的标签应该改成 `${someone}name`。

* **!** 用来强制把不存在的变量显示为空白

  当页面中包含 $msg，如果 msg 对象有值，将显示 msg 的值，如果不存在 msg 对象同，则在页面中将显示$msg 字符，这是我们不希望的。为了把不存在的变量或变量值为 null 的对象显示为空白，则只需要在变量名前加一个 **!** 号即可，如：`$!msg`

## 条件语句

```html
<!-- $foo 不为 null 或为 Boolean 对象的 false 值时执行. -->
#if ($foo) 
<strong>Velocity!</strong>
#end

#if ($bar)
Buy bar
#elseif ($car)
Buy car
#else
Buy tar
#end
```

## 循环语句

循环语句 `#foreach($var in $arrays) #end`，集合包含 Array, List, Set, Map

```html
<!-- 遍历 Array, List, Set -->
#foreach ($n in $ns)
$n<br>
#end

<!-- 遍历 map 的 value -->
#foreach ($value in $map)
$value<br>
#end

<!-- 遍历 map 的 entry set -->
#foreach ($entry in $map.entrySet())
$entry.getKey() - $entry.getValue()<br>
#end

#foreach ($customer in $customerList)
<tr><td>$velocityCount</td><td>$customer.Name</td></tr>
#end
```

> **$velocityCount** 是循环的计数器，默认从 1 开始，也可以修改配置从 0 开始
>
> Default name of the loop counter variable reference.
> `directive.foreach.counter.name = velocityCount`
>
> Default starting value of the loop counter variable reference.
> `directive.foreach.counter.initial.value = 1`

## 宏

宏的定义语法: `#macro(macroName [$param1 $param2 ...])`

```html
<!-- 定义宏: 无参数 -->
#macro(d)
<tr><td></td></tr>
#end

<!-- 调用宏 -->
#d()

<!-- 定义宏: 有参数 -->
#macro(tableRows $color $list)
    #foreach ($item in $list)
    <tr><td bgcolor=$color>$item</td></tr>
    #end
#end

<!-- 调用宏 -->
#tableRows($color $list)
```

## 包含文件

使用 **#include**  包含静态文件，不解析

```
#include("one.gif", "two.txt", "three.htm")
```

## 导入文件

使用 **#include** 导入 Velocity 模版文件，解析为 Velocity 脚本

```
#parse("products.vm")
```

## 不解析

`#[[don't parse me]]#` 的语法，允许模板设计者，在模板中方便地使用大块的无需翻译或解析的内容。

```
#[[
#foreach ($woogie in $boogie)
  对于$woogie，什么也不会发生
#end
]]#
```

输出

```
#foreach ($woogie in $boogie)  
  对于$woogie，什么也不会发生
#end
```

# stop

stop 停止执行 Velocity 脚本并返回

## 参考资料

更多详细的语法请参考 [Velocity User Guide 用户手册](http://blog.csdn.net/gaojinshan/article/details/23945879)


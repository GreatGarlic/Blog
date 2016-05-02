---
title: JavaScript 里格式化字符串
date: 2016-05-02 12:02:17
tags: [FE]
---

* 引入: `string-format.js`
* 使用: 
    * 参数为对象: `'http://www.xtuer.com/article/{id}'.format({id: 123})`
    * 参数为数组: `'http://www.xtuer.com/article/{0}'.format(123)`

<!--more-->

## string-format.js
```js
/**
 * 扩展了 String 类型，给其添加格式化的功能，替换字符串中 {placeholder} 或者 {0}, {1} 等模式部分为参数中传入的字符串
 * 使用方法:
 *     'I can speak {language} since I was {age}'.format({language: 'Javascript', age: 10})
 *     'I can speak {0} since I was {1}'.format('Javascript', 10)
 * 输出都为:
 *     I can speak Javascript since I was 10
 *
 * @param replacements 用来替换 placeholder 的 JSON 对象或者数组
 */
String.prototype.format = function(replacements) {
    replacements = (typeof replacements === 'object') ? replacements : Array.prototype.slice.call(arguments, 0);
    return formatString(this, replacements);
}

/**
 * 替换字符串中 {placeholder} 或者 {0}, {1} 等模式部分为参数中传入的字符串
 * 使用方法:
 *     formatString('I can speak {language} since I was {age}', {language: 'Javascript', age: 10})
 *     formatString('I can speak {0} since I was {1}', 'Javascript', 10)
 * 输出都为:
 *     I can speak Javascript since I was 10
 *
 * @param str 带有 placeholder 的字符串
 * @param replacements 用来替换 placeholder 的 JSON 对象或者数组
 */
var formatString = function (str, replacements) {
    replacements = (typeof replacements === 'object') ? replacements : Array.prototype.slice.call(arguments, 1);

    return str.replace(/\{\{|\}\}|\{(\w+)\}/g, function(m, n) {
        if (m == '{{') { return '{'; }
        if (m == '}}') { return '}'; }
        return replacements[n];
    });
};
```

## test.html
```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src="string-format.js"></script>
    <script>
        console.log(formatString("I can speak {language} since I was {age}", {language: 'Javascript', age: 10}));
        console.log(formatString("I can speak {0} since I was {1}",'Javascript', 10));

        console.log('I can speak {language} since I was {age}'.format({language: 'Javascript', age: 10}));
        console.log('I can speak {0} since I was {1}'.format('Javascript', 10));

        console.log('Unreplaced {{id}}'.format(123));
    </script>
</head>
<body>

</body>
</html>
```

## 输出
> I can speak Javascript since I was 10  
> I can speak Javascript since I was 10  
> I can speak Javascript since I was 10  
> I can speak Javascript since I was 10  
> Unreplaced {id}

---
title: JavaScript Tips
date: 2016-06-25 12:17:42
tags: FE
---

## true and false
>  '0' 不是 false  
>  ' ' 不是 false  
>  'false' 不是 false

<!--more-->

false 为:

* 空字符串 ''
* 数字 0
* false
* null
* undefined

true 则与上面的 false 相反

```js
var b1 = Boolean("");           // 返回 false, 空字符串 
var b2 = Boolean("s");          // 返回 true,  非空字符串 
var b3 = Boolean(0);            // 返回 false, 数字 0 
var b4 = Boolean(1);            // 返回 true,  非 0 数字 
var b5 = Boolean(-1);           // 返回 true,  非 0 数字 
var b6 = Boolean(null);         // 返回 false 
var b7 = Boolean(undefined);    // 返回 false 
var b8 = Boolean(new Object()); // 返回 true,  对象
```

## 字符串转换为数字
> 第一个非空白字符如果是非数字符则返回 NaN  
> 数字后面的非数字字符会被忽略掉

* parseInt(string, radix)，返回对应的 10 进制的值

    ```js
    parseInt("10");     //返回 10
    parseInt("19", 10); //返回 19 (10+9)
    parseInt("11", 2);  //返回 3 (2+1)
    parseInt("17", 8);  //返回 15 (8+7)
    parseInt("1F", 16); //返回 31 (16+15)
    parseInt("010");    //未定：返回 10 或 8
    parseInt("xyz");    // NaN
    parseInt('   10  xyz  '); // 10
    ```

* parseFloat(string)，返回一个浮点数

    ```js
    parseFloat("3.14");
    parseFloat("314e-2");
    parseFloat("0.0314E+2");
    parseFloat("3.14more non-digit characters"); // 3.14
    parseFloat("FF2"); // NaN
    ```

## 使用 Number 转换字符串为数字
> Number() 与 parseInt() 和 parseFloat() 类似，区别在于 Number() 转换的是整个值，而parseInt() 和 parseFloat() 则可以只转换开头的数字部分。Number() 会先判断要转换的值能否被完整的转换，然后再判断是调用 parseInt() 或 parseFloat()。

```js
Number(false);        // 返回 0 
Number(true);         // 返回 1 
Number(undefined);    // 返回 NaN 
Number(null);         // 返回 0 
Number("1.2");        // 返回 1.2 
Number("12");         // 返回 12 
Number(123);          // 返回 123 
Number("1.2.3");      // 返回 NaN 
Number(new Object()); // 返回 NaN 
```

## 使用 || 运算符实现默认参数
> 如果第一个参数返回的值为 false，那么第二个值将会认为是一个默认值

```js
function User(name, age) {
    this.name = name || "Oliver Queen";
    this.age = age || 27;
}
```

## 使用 === 而不是 ==
* `==`（或 `!=`）操作符在需要的时候会自动执行类型转换。
* `===`（或 `!==`）操作不会执行任何转换，它将比较值和类型，而且在速度上也被认为优于 `==`。

## 带有命名空间的类定义
```js
<!DOCTYPE html>
<html>

<head>
    <title></title>
</head>

<body>
    <script type="text/javascript">
        /**
         * 把字符串路径生成链式对象的命名空间。
         *
         * 使用命名空间，可以减少命名冲突，可以像下面这么做:
         * var ns = ns || {};
         * ns.ModuleClass = {};
         *
         * 但是如果名字太长，如 a.b.c.e.f.g.h，这样的方式定义命名空间需要太多代码，很麻烦。
         */
        function namespace(qualifiedPath) {
            var arr = qualifiedPath.split('.');
            var i = 0;
            var name;
            var root = window;

            while (name = arr[i++]) {
                if (!root[name]) {
                    root[name] = {};
                }

                root = root[name];
            }

            return root;
        }
    </script>

    <script>
        namespace('com.xtuer.util'); // 生成命名空间 com.xtuer.util

        // 定义类 Foo 的构造函数
        com.xtuer.util.Foo = function() {
            this.x = 10; // 定义成员变量
            this.y = 20; // 定义成员变量
        }

        // 给类 Foo 定义函数 bar()
        com.xtuer.util.Foo.prototype.bar = function() {
            console.log('[' + this.x + ', ' + this.y + ']'); // 调用成员变量
            this.foo(); // 调用成员函数
        }

        com.xtuer.util.Foo.prototype.foo = function() {
            console.log('Amazing');
        };

        var x = new com.xtuer.util.Foo(); // 创建对象，调用对象的函数
        x.bar();
        x.foo();
    </script>
</body>

</html>
```

## 删除字符串头尾的空白字符
* 使用 jQuery: `$.trim(string)`
* 自定义 trim() 函数: 

    ```js
    String.prototype.trim = function(){return this.replace(/^\s+|\s+$/g, "");};
    ```

## 将 arguments 对象转换成一个数组
```js
var argArray = Array.prototype.slice.call(arguments);
```

## 四舍五入一个数字，保留 N 位小数
```js
var num =2.443242342;
num = num.toFixed(4);  // num will be equal to 2.4432
```

## 使用 for-in 遍历一个对象内部属性
```js
for (var name in object) {
    if (object.hasOwnProperty(name)) { // 避免在遍历一个对象属性的时候访问原型的属性
        // do something with name
    }
}
```

## 基于 JSON 的序列化和反序列化
```js
var person = {name :'Saad', age : 26, department : {ID : 15, name : "R&D"} };
var stringFromPerson = JSON.stringify(person);
// stringFromPerson is equal to: 
// {"name":"Saad","age":26,"department":{"ID":15,"name":"R&D"}}

var personFromString = JSON.parse(stringFromPerson);
// personFromString is equal to person object
```

## 创建数组
```js
var ns = [];
```

## 对 URL 进行编码
```js
var myOtherUrl = "http://example.com/index.html?url=" + encodeURIComponent(myUrl);
```

## encodeURI 和 encodeURIComponent
它们都是用来对 URI 字符串进行编码的全局函数，但是它们的处理方式和使用场景有所不同。URI 中有几类不同的字符:

* 基本字符: 包括所有的大写字母、小写字母和数字 
* 保留字符: `/` `?` `:` `@` `&` `=` `+` `$` `,` `;`
* Mark 字符: `-` `_` `.` `!` `~` `*` `'` `(` `)`

使用场景

* `encodeURI`: 该函数对传入字符串中的所有非 `基本字符`、 `保留字符` 和 `Mark 字符` 进行转义编码，例如中文、字符空格(` ` 转换成为 `%20`)，用来对完整的 URI 字符串进行编码处理

* `encodeURIComponent`: 和 encodeURI 只有一个不同点，那就是对于 `保留字符` 同样做转义编码，例如，字符 `:` 被转义字符 `%3A` 代替，例如编码 URL 的参数列表中的每一个部分时使用

```js
encodeURI('http://www.xtuer.com/foo?username=中文&age=22');
// http://www.xtuer.com/foo?username=%E4%B8%AD%E6%96%87&age=22
// 没问题

encodeURIComponent('http://www.xtuer.com/foo?username=中文&age=22');
// http%3A%2F%2Fwww.xtuer.com%2Ffoo%3Fusername%3D%E4%B8%AD%E6%96%87%26age%3D22
// 有问题

encodeURI('http://www.xtuer.com/foo?redirect-url=http://www.xtuer.com/products&username=Bibo');
// http://www.xtuer.com/foo?redirect-url=http://www.xtuer.com/products&username=Bibo
// redirect-url 部分有问题，对 redirect-url 应该使用 encodeURIComponent 来编码
// 参数值中不应该有 : / 等

encodeURIComponent('http://www.xtuer.com/foo?redirect-url=http://www.xtuer.com/products&username=Bibo');
// http%3A%2F%2Fwww.xtuer.com%2Ffoo%3Fredirect-url%3Dhttp%3A%2F%2Fwww.xtuer.com%2Fproducts%26username%3DBibo
// 整个 URL 有问题
```

## 数组排序

```js
var ns = [5, 2, 3, 1, 4];
ns.sort(function(a, b) { // 自定义比较函数，可以对复杂对象进行比较
    return a - b;
});

console.log(ns); // 输出: [1, 2, 3, 4, 5]
```

---
title: PHP 快速入门
date: 2017-04-02 20:23:38
tags: PHP
---
`开发环境`：可以使用 `MAMP` [Download](http://www.mamp.info)

`经典的 Hello World`

```php
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

<body>

<?php
echo "Hello World!";
?>

</body>
</html>
```

* PHP 脚本可放置于文档中的任何位置
* PHP 脚本以 `<?php` 开头，以 `?>` 结尾
* 语句以分号结尾 `;`
* 注释：`# 单行注释`，`// 单行注释`，`/* 多行注释 */`
* 变量名对大小写敏感：`$color` 和 `$Color` 是不同的变量
* `用户定义的函数`、`类`和`关键字`（例如 if、else、echo 等等）都对大小写不敏感：`Echo "Ok"` 和 `echo "Ok"` 是一样的效果
* 输出内容到网页上用 `echo`
* `var_dump()`：会返回变量的数据类型和值，调试的时候很有用: var_dump("text"): `string(4) "text"`;
* `print_r`：Prints human-readable information about a variable

<!--more-->

## PHP 变量规则：
* 变量以 `$` 符号开头，其后是变量的名称
* 变量名称必须以字母或下划线开头
* 变量名称不能以数字开头
* 变量名称只能包含字母数字字符和下划线（A-z、0-9 以及 _）
* 变量名称对大小写敏感（$y 与 $Y 是两个不同的变量）

## 创建变量
* 创建变量的时候不指定变量的类型
* 变量没有类型
* PHP 自动的根据变量的值转换为正确的数据类型
* 变量会在首次赋值时被创建

```php
<?php
$x = 10; // 这样就创建好了一个变量
?>
```

## 创建常量: 使用 define() 函数创建常量 - 它使用三个参数：
* 首个参数定义常量的名称
* 第二个参数定义常量的值
* 可选的第三个参数规定常量名是否对大小写敏感。默认是 false(大小写敏感)

```php
<?php
define("TIME_ZONE", "Beijin");
echo TIME_ZONE; // Beijin
?>
```
## 变量的作用域
* `global`：`函数之外声明`的变量拥有 global 作用域，只能在函数以外进行访问
* `local` ：`函数内部声明`的变量拥有 local 作用域，只能在函数内部进行访问
* `static`：变量在函数执行完后不会被删除，就像 C 语言中用 static

`$GLOBALS[index]` 的数组中存储了所有的全局变量，下标为变量名

```php
<?php
$x = 10; // 全局变量
$y = 20; // 全局变量

function foo() { // 函数定义
    global $x; // 函数内访问全局变量需要在全局变量前加 global 关键字
    $z = 30;   // 局部变量

    // 输出：[x: 10, y: , z: 30]，函数内不能访问全局变量
    echo "[x: $x, y: $y, z: $z]<br>";
    
    // 用 GLOBAL 数组访问全局变量  
    echo "x: {$GLOBALS['x']} <br>";
}

foo(); // 执行函数

// 输出：[x: 10, y: 20, z: ]，函数外不能访问局部变量
echo "[x: $x, y: $y, z: $z]<br>"; 

/////////////////////////////////////////////////////////////////////////

function bar() {
    static $x = 0;
    echo "static: $x <br>";
    $x++;
}

bar(); // static: 0 
bar(); // static: 1
bar(); // static: 2
?>
```

## 运算符
`+` `-` `*` `/` `%` 
`>` `<` `>=` `<=` `!=` `==` `===` `||` `&&`
`前置++` `后置++` `前置--` `后置--` 
`.` `.=` 用于字符串连接

比较特别的是 `==`(值相同就认为相等) `===`(值相同，并且类型也要相同)

```php
<?php
$x = 10;
$y = "10";

var_dump($x); // int(10)
echo "<br>";

var_dump($y); // string(2) "10"
echo "<br>";

echo $x == $y;  // 1
echo "<br>";
echo $x === $y; // false 输出是空字符串
echo "<br>";
echo ($x === $y) == false;  // 1
echo "<br>";
echo ($x === $y) === false; // 1


echo "<br>";
echo $x . " apples"; // 10 apples
?>
```

## 数字与字符串比较
数字与字符串比较时, `先尝试将字符串转换为数字`, 再比较, 一个不能转换为数字的字符串, 转换结果为0, 故, 与 0 比较总返回 true

```php
<?php
// String to integer
// 返回字符串中第一个不是数字的字符之前的数字串所代表的整数值。
// 如果字符串第一个是‘-'，则从第二个开始算起
echo (int)("a11"); // 0
echo "<br>";
echo (int)("11a"); // 11
echo "<br>";
echo intval("a11"); // 0
echo "<br>";
echo intval("11a"); // 0
echo "<br>";
echo intval("11", 2); // 3
?>
```

## 控制语句
`if else` `for` `foreach` `while` `do while` `switch case`

```php
<?php
///////////////////////////////////////////////////////////////////////////////
// if else
///////////////////////////////////////////////////////////////////////////////
echo "<br><br>If else<br>";
$score = 70;

if ($score < 60) {
    echo "小于 60 <br>";
} else if ($score < 80) {
    echo "大于等于 60，小于 80 <br>";
} else {
    echo "大于 80 <br>";
}

///////////////////////////////////////////////////////////////////////////////
// for: 输出 1 到 10
///////////////////////////////////////////////////////////////////////////////
echo "<br><br>For<br>";

for ($i = 1; $i <= 10; $i++) {
    echo "$i<br>";
}

///////////////////////////////////////////////////////////////////////////////
// foreach: 输出数组中的所有元素
///////////////////////////////////////////////////////////////////////////////
// foreach 循环只适用于数组，并用于遍历数组中的每个键/值对。
echo "<br><br>Foreach<br>";

$colors = array("Red", "Green", "Blue", "White", "Black");

foreach ($colors as $color) {
    echo "$color <br>";
}

///////////////////////////////////////////////////////////////////////////////
// while: 输出 1 到 10
///////////////////////////////////////////////////////////////////////////////
echo "<br><br>While<br>";
$i = 0;

while ($i++ < 10) {
    echo "$i<br>";
}

///////////////////////////////////////////////////////////////////////////////
// do while: 输出 1 到 10
///////////////////////////////////////////////////////////////////////////////
echo "<br><br>Do while<br>";
$i = 1;

do {
    echo "$i<br>";
} while (++$i <= 10);

///////////////////////////////////////////////////////////////////////////////
// switch: 可以使用字符串，数字等
///////////////////////////////////////////////////////////////////////////////
echo "<br><br>Switch<br>";

$condition = "A";

switch ($condition) {
case 1:
    echo "Is 1 <br>";
    break;
case "a":
case "A":
    echo "Is A <br>";
    break;
case "B":
    echo "Is B <br>";
    break;
default:
    echo "Default <br>";
}
?>
```

## 函数
PHP 内置了 1000 多个函数，功能很强大，例如要计算一个字符串的 MD5 并转换为大写 `strtoupper(md5("Tidy Code"))` 得到 `ADCF1E98EBD0FD99E1227346B70BD9E2`。

函数创建和 JavaScript 很像，都是以关键字 `function` 开头，然后是`函数名`和`参数列表`，`参数支持默认值`，函数的定义和`调用顺序`没有要求，可以递归调用。

```php
<?php
echo fibonacc(10) . "<br>"; // 函数调用，输出：55

/**
 * 函数定义: 递归实现斐波那契数列
 */
function fibonacc($n) { // 变量 $n 的作用域 是 local.
    if ($n == 1 || $n == 2) { // 递归结束条件
        return 1;
    }

    return fibonacc($n - 1) + fibonacc($n - 2); // 递归
}

// 输出：1, 1, 2, 3, 5, 8, 13, 21, 34, 55,
for ($i = 1; $i <= 10; $i++) {
    echo fibonacc($i) . ", ";
}

echo "<br>";

/**
 * 默认参数只能在参数列表最后面，可以有多个默认参数
 */
function foo($name, $email = "xxxx@gmail.com", $mobile = "xxxxxxxxxxx") {
    echo "Name is: $name, EMail is: $email, Mobile is: $mobile <br>";
}

// 输出：Name is: Alice, EMail is: alice@salmon.com, Mobile is: 12345678901
foo("Alice", "alice@salmon.com", "12345678901");

// 输出：Name is: Alice, EMail is: alice@salmon.com, Mobile is: xxxxxxxxxxx
foo("Alice", "alice@salmon.com");

// 输出：Name is: Alice, EMail is: xxxx@gmail.com, Mobile is: xxxxxxxxxxx
foo("Alice");
?>
```

## 数组
`PHP 里的数组`实际上是一个字典 `Dictionary`，也可以叫 `Map`，就是用 `key/value` 的形式存储。没有给出 key 的时候 key 默认就是用下标 0，1，2 等。

数组用 `array()` 来创建，数组的长度用 `count()` 来计算。

> 同一个 `array` 对象推荐单纯的作为`数组`使用，或者是 `Map` 使用。
> 不要即有数组的用法，同时也有 Map 的用法。

### `作为普通数组使用`
```php
<?php
$colors = array("Red", "Green", "Blue"); // 定义数组

///////////////////////////////////////////////////////////////////////////////
// 使用下标直接访问数组元素
///////////////////////////////////////////////////////////////////////////////
$colors[0] = "Yellow"; // 给数组赋元素值

echo "$colors[0], $colors[1], $colors[2] <br>"; // 用下标访问数组的元素

echo "-------------------------------------------<br>";
///////////////////////////////////////////////////////////////////////////////
// 使用 for 循环遍历数组
///////////////////////////////////////////////////////////////////////////////
$len = count($colors);

for ($i = 0; $i < $len; $i++) {
    echo $colors[$i];
    echo "<br>";
}

echo "-------------------------------------------<br>";
///////////////////////////////////////////////////////////////////////////////
// 使用 foreach 循环遍历数组
///////////////////////////////////////////////////////////////////////////////
foreach ($colors as $color) {
    echo $color;
    echo "<br>";
}
?>
```

`使用数组排序`

```php
<?php
$numbers = array(1, 3, 5, 9, 5, 8, 4);
$len = count($numbers);

echo join(", ", $numbers) . "<br>"; // 输出数组元素

// 选择排序: 升序
for ($i = 0; $i < $len - 1; $i++) {
    $k = $i;
    for ($j = $i; $j < $len; $j++) {
        if ($numbers[$j] < $numbers[$k]) {
            $k = $j;
        }
    }

    $temp = $numbers[$k];
    $numbers[$k] = $numbers[$i];
    $numbers[$i] = $temp;
}

echo join(", ", $numbers) . "<br>";
?>
```

### `作为 Map 使用`
```php
<?php
$ages = array("Alice"=>"20", "Bob"=>"25", "Josh"=>"30"); // 创建 Map

///////////////////////////////////////////////////////////////////////////////
// 使用 key 访问 value，就像 Java 的 Map： map.getValue(key)
///////////////////////////////////////////////////////////////////////////////
echo $ages["Alice"];
echo "<br>";

echo "-------------------------------------------<br>";
///////////////////////////////////////////////////////////////////////////////
// 输出所有的 key 和 value.
///////////////////////////////////////////////////////////////////////////////
foreach($ages as $key=>$value) {
    echo "Key: $key, Value: $value";
    echo "<br>";
}

echo "-------------------------------------------<br>";
///////////////////////////////////////////////////////////////////////////////
// 输出所有的 value
///////////////////////////////////////////////////////////////////////////////
foreach($ages as $value) {
    echo "Value: $value";
    echo "<br>";
}
?>
```

### 作为数组和 Map 混用，`下标`和 `key` 理解起来就比较混乱
```php
<?php
$colors = array("Red", "Green", "Blue"); // 下标是 0，1，2，实际应该理解为 key。
$colors[5] = "Yellow";
$colors[6] = "Purple";
$colors["pink"] = "Pink"; // 第 5 个元素，但是不能用 $colors[5] 访问(是 Yellow)

$len = count($colors);

for ($i = 0; $i < $len; ++$i) {
    echo $i . "  " . $colors[$i]; // 下标 3，4 没有元素，所以输出 到 3，4 时会报错
    echo "<br>";
}

echo "-------------------------------------------<br>";
foreach ($colors as $color) {
    echo $color;
    echo "<br>";
}

echo "-------------------------------------------<br>";
echo $colors[5]; // 数字为 key 时可以不用引号
echo "<br>";
echo $colors["5"];
?>
```

## 处理 GET 和 POST 请求
使用 `$_GET["fieldName"]` 取得 get 请求的数据
使用 `$_POST["fieldName"]` 取得 post 请求的数据

```php
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
    <form action="form.php" method="GET"> <!-- 换成 POST -->
        Name: <input type="text" name="name">
        Mail: <input type="text" name="mail">
        <input type="submit" value="Submit">
    </form>
    <hr>

<?php
$name = $_GET["name"]; // 换成 $_POST
$mail = $_GET["mail"];

echo "Name is: $name, Mail is: $mail";
?>

<hr>

<?php
$name = "";
$mail = "";

if ($_SERVER["REQUEST_METHOD"] == "GET") { // GET or POST
    $name = $_GET["name"];
    $mail = $_GET["mail"];
} else {
    $name = $_POST["name"];
    $mail = $_POST["mail"];
}

echo "Request method: {$_SERVER["REQUEST_METHOD"]}<br>";
echo "Name is: $name, Mail is: $mail";
?>
</body>
</html>
```

## Cookie
`设置 cookie`: setcookie(name, value, expire, path, domain)
`取得 cookie`: $\_COOKIE[name]
`删除 cookie`: setcookie(name, "", time()-3600) 设置 cookie 过期就可以了
`测试 cookie`: isset($\_COOKIE[name])

### `设置 cookie 的页面`
```php
<?php
setcookie("user", "Bob", time() + 3600); // 必须在 html 的内容前面设置 cookie
?>

<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

<body>
</body>
</html>
```

### `访问 cookie 的页面`
```php
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>

<?php
echo $_COOKIE["user"];
echo "<br>";

if (isset($_COOKIE["mail"])) {
    echo $_COOKIE["mail"];
}

?>
</body>
</html>
```

## Session
使用 session 前必须`启动 session`: session\_start()
`设置 session`: $\_SESSION[name]=value
`取得 session`: $\_SESSION[name]
`删除 session`: unset($\_SESSION[name])，session_destroy()
`测试 session`: isset($\ _SESSION[name])

### `设置 session 的页面`
```php
<?php
session_start(); // 必须在 html 的内容前面设置 cookie
$_SESSION['number']=1;
?>

<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

<body>
</body>
</html>
```

### `访问 session 的页面`
```php
<?php
session_start(); // 必须在 html 的内容前面设置 cookie
?>

<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>

<?php
echo $_SESSION["number"];
?>
</body>
</html>
```

## Include / Require 文件
> 通过 include 或 require 语句，可以将 PHP 文件的内容插入另一个 PHP 文件（在服务器执行它之前）。
> include 和 require 语句是相同的，除了错误处理方面：
>
* require 会生成致命错误（E\_COMPILE\_ERROR）并停止脚本
* include 只生成警告（E_WARNING），并且脚本会继续
>
> 因此，如果您希望继续执行，并向用户输出结果，`即使包含文件已丢失，那么请使用 include`。否则，在框架、CMS 或者复杂的 PHP 应用程序编程中，请始终使用 require 向执行流引用关键文件。这有助于提高应用程序的安全性和完整性，在某个关键文件意外丢失的情况下。
>包含文件省去了大量的工作。这意味着您可以为所有页面创建标准页头、页脚或者菜单文件。然后，在页头需要更新时，您只需更新这个页头包含文件即可。

`include 'filename';`
`require 'filename';`
`include_once("fileName");`
`require_once("fileName");` <mark>括号可要可不要</mark> <mark>类的定义推荐用 require_once</mark>

```php
<!-- footer.php -->
<?php
echo "<p>Copyright © 2006-" . date("Y") . " W3School.com.cn</p>";
?>
```

```php
<!-- hello.php -->
<html>
<body>
    <?php include 'header1.php';?></body>
    <h1>Hello World!</h1>
    <p>Section 1</p>
    <p>Section 2</p>
    <?php require 'footer.php';?></body>
    <p>End</p>
</html>
```

## PHP 的面向对象：类，和 Java 的类很像
> 
* 类的定义：`class className {}` <mark>没有访问权限一说</mark>
* 成员变量定义：`[qualifier|var] $field;` <mark>可以初始化</mark>
* 成员函数定义：`[qualifier] function methodName() {}`，<mark>可带参数</mark>
* 构造函数：`__construct() {}`，<mark>可带参数</mark>
* 创建对象：`$obj = new className();`，`$obj = new className($v1,$v2);`
* 函数调用：`$obj->methodName();`
* 成员函数内访问成员变量：`$this->fieldName` <mark>不是 $this->$fieldName</mark>
* 成员函数内访问成员函数：`$this->methodName()`
* PHP 不会自动调用父类的构造函数(不支持构造函数重载，可以使用默认参数实现重载)，必须使用 `parent` 关键字显式地调用
* PHP 只支持`单继承`，使用关键字 `extends`
* PHP 也有`接口`，使用关键字 `interface` 定义接口，实现接口使用关键字 `implements`
* 接口的方法没有实现会报错
* 访问权限 `qualifier`：`public`, `protected`, `private`, `abstract`, `final`, `static`。`默认是 public` 的访问权限
* 可以把类的定义放在单独的文件里，然后使用 `require_once` 加载

### `类定义和继承`

```php
<html>
<body>
<?php
class Employee {
    var $name;
    var $salary;
    protected $id;

    function __construct($name, $salary) { // 构造函数
        $this->name = $name;
        $this->salary = $salary;
    }

    function getName() {
        return $this->name; // 不能直接用 return $name;
    }

    function setName($name) {
        $this->name = $name;
    }
}

$employee = new Employee("Alice", 2000);
echo $employee->getName() . "<br>";

$employee->setName("Bob");
echo $employee->getName() . "<br>";
?>

<?php
// 使用继承
class Manager extends Employee {
    var $title;

    function __construct($name, $salary, $title) {
        parent::__construct($name, $salary); // 调用父类的构造函数
        $this->title = $title;
    }

    function getTitle() {
        return $this->title;
    }
}

$manager = new Manager("Alice", 3000, "Project Manager");
echo $manager->getName() . ", " . $manager->getTitle() . "<br>";
?>
</body>
</html>
```

### `实现接口`
```php
<html>
<body>
<?php
interface Flyable {
    function fly();
}

interface Runnable {
    function run();
}

class Bird implements Flyable, Runnable {
    function fly() {
        echo "I can fly.<br>";
    }

    function run() {
        echo "I can run.<br>";
    }
}

$bird = new Bird();
$bird->fly();
$bird->run();
?>
</body>
</html>
```

### `静态访问` <mark>关键字 self</mark>
```php
<html>
<body>
<?php
class Test {
    public static $count = 0;

    // 每创建一个对象 count 就加 1
    function __construct() {
        self::$count++; // 函数内部访问静态成员
        echo self::$count . " instances are created.<br>";
    }

    static function getCount() {
        return self::$count;
    }
}

echo Test::getCount() . "<br>"; // 类外部访问静态成员函数
new Test();
new Test();
new Test();
echo Test::getCount() . "<br>";
?>
</body>
</html>
```

## 实用代码
### Array to JSON
```php
<?php
$props = array("background"=>"black", "width"=>300, "height"=>600);
echo json_encode($props); // {"background":"black","width":300,"height":600}
?>
```

### 取得正在访问的网页的网址
```php
<?php
$url = "http://".$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'];
echo $url . "<br>"; // http://localhost:8000/hello.php

$name = dirname($url);
echo $name . "<br>"; // http://localhost:8000
?>
```

### 使用 PHP 发邮件
```php
<html>
<head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8" />
</head>

<body>
    <form action="hello.php" method="post">
        <table>
            <tr>
                <td>Mail to:</td>
                <td><input type="text" name="mail_to" /></td>
            </tr>
            <tr>
                <td>Subject:</td>
                <td><input type="text" name="subject" /></td>
            </tr>
            <tr>
                <td>Mail From:</td>
                <td><input type="text" name="mail_from" /></td>
            </tr>
            <tr>
                <td colspan="2"><textarea name="contents" cols="50" rows="10"></textarea></td>
            </tr>
            <tr>
                <td colspan="2" align="center"><input type="submit" value="Send Mail"/></td>
            </tr>
        </table>
    </form>

    <?php
        $mailTo = $_POST["mail_to"];
        $mailFrom = $_POST["mail_from"];
        $subject = $_POST["subject"];
        $contents = $_POST["contents"];

        if ($mailTo && $mailFrom && $subject && $contents) {
            // 前面那些HTML代码都是为了填写信息更容易，实际发送邮件代码只是下面一句
            // 但首先机器上得启动邮件发送程序，Mac下是默认启动的，Linux好像要自己先配置启动
            mail($mailTo, $subject, $contents, "From:".$mailFrom);
            echo "Mail is successfully sent!", "<br>";
        }
    ?>
</body>
</html>
```

### PHP中的非贪婪匹配，默认用的是贪婪匹配
```php
<pre>
<?php
// 替换图片的目录为统一路径
$string = 'a<img src="a/b/x.png">,Biao,<img src="uploads/y.jpg">';
$string2 = 'a<img src="a/b/x.png">,Biao,<img src="uploads/y.jpg">';

// /U为非贪婪，这种非贪婪，是从后面向前找，与+?有些区别
$pattern ='/<img src="(.*)\/(.+\\..+)">/U';
$replacement = "<img src=\"__IMG_BASE__/$2\">";

echo preg_replace($pattern, $replacement, $string);
echo "-------------";

$pattern ='/<img src="(.*)\/(.+\\..+)">+?/'; // +?结果与上面的不一样

echo preg_replace($pattern, $replacement, $string2);
echo "-------------";

// 找出所有的图片，图片名
$pattern ="/<img src=\"(.*)\/(.+\\..+)\">/U";
preg_match_all($pattern, $string, $images);
print_r($images);

// 输出图片名
foreach ($images[2] as $index => $imageName) {
    echo "$imageName<br/>";
}
?>
```

### 字符串和timestamp的转换 [Date Document](http://php.net/manual/en/function.date.php)
```php
<?php
echo (strtotime("2010-10-28 10:52:21")); // MySQL格式的字符串转换成秒数
echo "<br/>";
echo (strtotime("2010-10-01 00:00:00"));
echo "<br/>";
$d = date("Y-m-d H:i:s", time()); // 秒数转换成MySQL格式的timestamp
echo $d;
?>
```

### 访问 MySQL
```php
<?php
// 插入数据到数据库中, 数据库操作语句都是使用mysql_query
mysql_connect("localhost", "root", "root"); // 连接到数据库: url, username, password
mysql_select_db("qt"); // 选择使用数据库中的表

$result = mysql_query("SELECT username, password FROM user");
while ($row = mysql_fetch_array($result)) { // 每使用一次后，会自动移向下一个游标
    // username 和 password 是数据库中列名
    echo $row["username"], ", ", $row["password"], "<br>";
}
?>

<?php
$username = "Blabla";
$password = date("H:i:s");

// 插入数据
mysql_query("INSERT INTO USER (username, password) VALUES ('$username', '$password')");

// Close MySQL connection
mysql_close();
?>
```

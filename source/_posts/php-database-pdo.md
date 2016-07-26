---
title: PHP 使用 PDO 访问数据库
date: 2016-07-26 11:42:45
tags: PHP
---

PHP 中可以使用 PDO 访问数据，PHP 5.0 后自带 PDO，PDO 还能防止 SQL 注入功能。

<!--more-->

## DBUtils.php
```php
<?php
class DBUtils {
    public static function query($sql, $params) {
        $results = array();

        try {
            // 数据库连接信息
            $db = new PDO("mysql:host=localhost;dbname=test", "root", "root", array(PDO::ATTR_PERSISTENT => true));

            $preparedStatement = $db->prepare($sql);
            $preparedStatement->execute($params);

            foreach ($preparedStatement as $row) {
                array_push($results, $row);
            }

            $db = null;
        } catch (PDOException $e) {
            print "Error: " . $e->getMessage() . "<br/>";
            die();
        }

        return $results;
    }
}
?>
```

## a.php
```php
<?php
// 开启错误输出
ini_set("display_errors", "On");
error_reporting(E_ALL | E_STRICT);

// 引入访问数据库的工具类
include_once "DBUtils.php";

// Named prepared SQL
$sql = "SELECT * FROM foo WHERE name=:name OR city=:city";

// 查询数据库
$results = DBUtils::query($sql, array(":name"=>"Biao", ":city"=>"Beijin"));
var_dump($results);
?>
```

## 测试
* 访问 <http://localhost//a.php>

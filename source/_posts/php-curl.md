---
title: PHP Curl
date: 2016-09-28 13:53:29
tags: PHP
---

PHP 使用 Curl 访问网址

<!--more-->

## GET
```php
<?php
// 要访问的网页的网址
$url = "http://192.168.10.49:8080/signIn";

// 请求的参数中可能会有特殊符号，中文等，需要对其进行编码，编码的操作在 curlGet 中已经执行
// 这样就不需要手动的编码了(还容易出错)
$params = array(
    "id" => 1,
    "name" => "Bob/Alice/桑迪"
);

echo curlGet($url, $params);

/**
 * 使用 Curl 执行 GET 请求
 *
 * @param  string  $url     请求的 URL
 * @param  array   $params  请求的参数
 * @param  integer $timeout 请求的超时
 * @return string           请求响应的结果
 */
function curlGet($url, $params, $timeout = 5) {
    if($url == "" || $timeout <= 0) {
        return "";
    }

    $url = $url.'?'.http_build_query($params);
    $con = curl_init((string)$url);

    curl_setopt($con, CURLOPT_HEADER, false);
    curl_setopt($con, CURLOPT_RETURNTRANSFER,true);
    curl_setopt($con, CURLOPT_TIMEOUT, (int)$timeout);
    $result = curl_exec($con);
    curl_close($con);

    echo $result;
}
?>
```

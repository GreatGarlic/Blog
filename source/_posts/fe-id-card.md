---
title: 解析身份证
date: 2016-10-20 14:26:31
tags: FE
---
身份证号码有 15 位和 18 位的，下面解释 18 位身份证号码不同部分数子的含意:

1. 前 `1、2` 位数字表示：所在`省份`的代码
2. 第 `3、4` 位数字表示：所在`城市`的代码
3. 第 `5、6` 位数字表示：所在`区县`的代码
4. 第 `7~14` 位数字表示：出生`年、月、日`
5. 第 `15、16` 位数字表示：所在地的`派出所`的代码(也有说不是，如图)
6. 第 `17` 位数字表示`性别`：奇数表示男性，偶数表示女性
7. 第 `18` 位数字是`校检码`：也有的说是个人信息码，用来检验身份证的正确性。校检码可以是 0~9 的数字，有时也用 x 表示(尾号是10，那么就得用 x 来代替)，一般是随计算机的随机产生

![](/img/fe/id-card.jpg)

<!--more-->

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>解析身份证</title>
    </head>
    <body>
        <script type="text/javascript">
            function IdCard(name, idNo) {
                this.name          = name;
                this.birthday      = idNo.substring(6, 15);
                this.birthdayYear  = this.birthday.substring(0, 4);
                this.birthdayMonth = this.birthday.substring(4, 6);
                this.birthdayDay   = this.birthday.substring(6, 8);
                this.gender        = (parseInt(idNo.substring(16, 17)) % 2 == 0) ? '女' : '男';
                this.genderValue   = (parseInt(idNo.substring(16, 17)) % 2 == 0) ? 1 : 2;
            }

            IdCard.validate = function(idNo) {
                return /^\d{17}[\dxX]$/.test(idNo);
            }

            console.log(JSON.stringify(new IdCard('黄晓明', '330726196507040016')));
            console.log(JSON.stringify(new IdCard('陶君华', '430421197710177894')));
            console.log(JSON.stringify(new IdCard('女性的', '110102198611267047')));

            console.log(IdCard.validate('110102198611267047')); // true
            console.log(IdCard.validate('11010219861126704x')); // true
            console.log(IdCard.validate('11010219861126704X')); // true
            console.log(IdCard.validate('11010219861126704a')); // false
        </script>
    </body>
</html>
```

输出:

```
{"name":"黄晓明","birthday":"196507040","birthdayYear":"1965","birthdayMonth":"07","birthdayDay":"04","gender":"男","genderValue":2}
{"name":"陶君华","birthday":"197710177","birthdayYear":"1977","birthdayMonth":"10","birthdayDay":"17","gender":"男","genderValue":2}
{"name":"女性的","birthday":"198611267","birthdayYear":"1986","birthdayMonth":"11","birthdayDay":"26","gender":"女","genderValue":1}
true
true
true
false
```


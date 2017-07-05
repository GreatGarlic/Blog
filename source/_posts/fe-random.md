---
title: 自定义随机函数
date: 2017-07-05 16:35:45
tags: FE
---

JavaScript 已经自带了随机数生成函数，为什么我们还需要弄一个随机数的生成工具呢？

例如 Web 的考试系统里，加载试卷后，需要把试卷的题目顺序打乱，如果用 JS 的随机数函数的话，每次打乱的顺序都是不一样的，因为每次生成的随机数序列都不一样。问题出现了，同一个学员刷新试卷后，题目的顺序和上一次的竟然不一样，这不符合实际要求，应该是不同学员的题目顺序不一样，但是同一个学员的题目顺序永远是一样的。这样就不能使用直接使用原生的随机函数了，下面定义一个随机数生成函数，随机数种子是一个字符串，这样就可以用学员的编码来作为随机数种子生成随机数打乱题目的顺序了，因为每次刷新时同一个学员的编码都是一样的，所以生成的随机数序列都相同，就保证了同一个学员的试卷题目顺序一直都是一样的。

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
</head>

<body>
    <script type="text/javascript">
        /**
         * 生成随机数的类，参数为随机数的种子，如果种子相同，则生成的随机数序列一样。
         *
         * @param {String 或者 Integer} seed 字符串或者整数的随机数种子
         */
        function Random(seed) {
            this.seed = this.hashCode(seed + '');
        }

        Random.prototype.hashCode = function(str) {
            var hash = 0;

            if (str.length == 0) return hash;

            for (i = 0; i < str.length; i++) {
                char = str.charCodeAt(i);
                hash = hash * 31 + char;
                hash = hash & hash; // Convert to 32bit integer
            }

            return hash;
        }

        /**
         * 生成 0 到 1 千万之间的随机整数
         *
         * @return {Integer} 返回一个随机整数
         */
        Random.prototype.nextInt = function() {
            this.seed = (this.seed * 9301 + 49297) % 233280;
            var t = this.seed / 233280.0;

            return Math.abs(Math.ceil(t * 10000000));
        }

        /*******************************************************************************
         *                                 Test random                                 *
         ******************************************************************************/
        // 测试生成随机数
        var random = new Random(103); // 使用学号作为种子创建随机数对象

        for (var i = 0; i < 100; ++i) {
            console.log(i + ': ' + random.nextInt() % 80);
        }

        // 测试生成的随机数分布是否均衡：结果还是比较均衡的
        var max = 10;
        var frequence = {};
        for (var i = 0; i < max; ++i) {
            frequence['' + i] = 0;
        }
        for (var i = 0; i < 10000; ++i) {
            var r = random.nextInt() % max;
            frequence['' + r] = frequence['' + r] + 1;
        }
        console.log(frequence);
    </script>
</body>

</html>
```


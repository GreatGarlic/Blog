---
title: Vue 的省市多级联动效果
date: 2016-12-01 11:06:30
tags: FE
---
使用 Vue 实现多级连动效果，需要了解 

* `v-model`
* 属性绑定 `:value`
* change 事件处理 `@change`
* Ajax 请求 `$.getJSON()`

![](/img/fe/vue-chain.png)

<!--more-->

## 前端代码

当省变化时，清空所有市，县，乡的数据，然后请求市的数据  
当市变化时，清空所有县，乡的数据，然后请求县的数据  
当县变化时，清空所有乡的数据，然后请求乡的数据

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>测试</title>
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.min.js"></script>
    <style media="screen">
        body {
            font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
        }
    </style>
</head>
<body>
    <!-- Vue 使用的模版 -->
    <div id="vue-app">
        <div class="">
            省:
            <select v-model="selectedProvinceId" @change="provinceChanged(selectedProvinceId)">
                <option v-for="province in provinces" :value="province.id">{{province.name}}</option>
            </select>
        </div>
        <div class="">
            市:
            <select v-model="selectedCityId" @change="cityChanged(selectedCityId)">
                <option v-for="city in cities" :value="city.id">{{city.name}}</option>
            </select>
        </div>
        <div class="">
            县:
            <select v-model="selectedCountryId" @change="countryChanged(selectedCountryId)">
                <option v-for="country in countres" :value="country.id">{{country.name}}</option>
            </select>
        </div>
        <div class="">
            乡:
            <select v-model="selectedVillageId">
                <option v-for="village in villages" :value="village.id">{{village.name}}</option>
            </select>
        </div>
    </div>

    <script>
        $(document).ready(function() {
            // 使用 Vue
            var app = new Vue({
                el: '#vue-app',
                data: {
                    selectedProvinceId: 0,
                    selectedCityId: 0,
                    selectedCountryId: 0,
                    selectedVillageId: 0,

                    provinces: [], // 省
                    cities: [], // 市
                    countres: [], // 县
                    villages: [] // 街道
                },
                methods: {
                    provinceChanged: function(provinceId) {
                        this.cities = [];
                        this.countres = [];
                        this.villages = [];

                        // 请求市
                        $.getJSON('http://localhost:8080/areas-of/' + provinceId, function(cities) {
                            app.cities = cities;
                        });
                    },
                    cityChanged: function(cityId) {
                        this.countres = [];
                        this.villages = [];

                        // 请求县
                        $.getJSON('http://localhost:8080/areas-of/' + cityId, function(countres) {
                            app.countres = countres;
                        });
                    },
                    countryChanged: function(countryId) {
                        this.villages = [];

                        // 请求乡
                        $.getJSON('http://localhost:8080/areas-of/' + countryId, function(villages) {
                            app.villages = villages;
                        });
                    }
                }
            });

            // 请求省
            $.getJSON('http://localhost:8080/areas-of/0', function(provinces) {
                app.provinces = provinces;
            });
        });
    </script>
</body>
</html>
```

## 请求响应数据格式

```json
[
    {
        "id":4813,
        "name":"敦煌市",
        "level":3,
        "parentId":456
    },
    {
        "id":4814,
        "name":"玉门市",
        "level":3,
        "parentId":456
    },
    {
        "id":4815,
        "name":"瓜州县（原安西县）",
        "level":3,
        "parentId":456
    },
    {
        "id":4816,
        "name":"肃北蒙古族自治县",
        "level":3,
        "parentId":456
    },
    {
        "id":4817,
        "name":"肃州区",
        "level":3,
        "parentId":456
    },
    {
        "id":4818,
        "name":"金塔县",
        "level":3,
        "parentId":456
    },
    {
        "id":4819,
        "name":"阿克塞哈萨克族自治县",
        "level":3,
        "parentId":456
    }
]
```



## 数据库表结构

| id   | name | parent_id | level |
| ---- | ---- | --------- | ----- |
| 1    | 北京市  | 0         | 1     |
| 2    | 天津市  | 0         | 1     |
| 3    | 河北省  | 0         | 1     |
| 37   | 东城区  | 1         | 2     |
| 38   | 西城区  | 1         | 2     |
| 987  | 大寺镇  | 65        | 3     |

省市县等的数据下载: [area.sql](/download/area.sql)

## 服务器端代码

服务器端使用 SpringMvc + MyBatis 实现

### Area

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Area {
    private int id;
    private String name;
    private int level;
    private int parentId;
}
```

### AreaController

```java
package com.xtuer.controller;

import com.xtuer.bean.Area;
import com.xtuer.mapper.AreaMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
public class AreaController {
    @Autowired
    private AreaMapper areaMapper;

    @GetMapping("/areas-of/{parentId}")
    @ResponseBody
    public List<Area> findAreasByParentId(@PathVariable int parentId) {
        return areaMapper.findAreasByParentId(parentId);
    }
}
```

### AreaMapper

```java
package com.xtuer.mapper;

import com.xtuer.bean.Area;

import java.util.List;

public interface AreaMapper {
    List<Area> findAreasByParentId(int parentId);
}
```

## Area.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC
        "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace 非常重要：必须是 Mapper 类的全路径-->
<mapper namespace="com.xtuer.mapper.AreaMapper">
    <select id="findAreasByParentId" parameterType="int" resultType="Area">
        SELECT id, name, level, parent_id AS parentId
        FROM area
        WHERE parent_id=#{parentId}
    </select>
</mapper>
```

## 思考

**上面的代码有问题没？**

有，大大的有，服务器端和浏览器端都有优化的空间

* 服务器端: 上面的实现，每次数据都要从数据库里查询，并发量大的时候，由于数据库访问太多，系统的性能会急剧下降，甚至会导致系统不响应。观察地区数据的特点，他们是很少变化的，是不是可以先从缓存里读取，如果缓存里没有，再从数据库读取？可参考 [Redis 集成](/spring-web-redis)
* 浏览器端: 例如先选择北京，加载北京下的数据，然后选择河北，加载河北下的数据，再选择北京，又向服务器请求加载北京的数据，发现向服务器请求了 2 次北京的数据，可以在浏览器端对请求到的数据缓存，同一个数据不要进行多次加载，在访问量很大的系统中，每一个小的优化都很关键


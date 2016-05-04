---
title: Solr Tips
date: 2016-04-19 20:42:10
tags: [Java, Solr]
---

## 安装 Solr
* 下载 Solr: <http://www.apache.org/dyn/closer.lua/lucene/solr/6.0.0>
* 解压到任意目录，不需要安装，绿色软件

<!--more-->

## 准备 Solr
* 启动 Solr: `bin/solr start`
* 创建 Core，命名为 ebag: `bin/solr create -c ebag`
    * `-c` 的意义为 Core or Collection，不可缺少
* 添加索引: `bin/post -c ebag example/exampledocs/books.json`
* 搜索: <http://localhost:8983/solr/ebag/select?indent=on&wt=json&q=Greek>，输出

    ```js
    {
        "responseHeader": {
            "zkConnected": true,
            "status": 0,
            "QTime": 1,
            "params": {
                "q": "Greek",
                "indent": "on",
                "wt": "json"
            }
        },
        "response": {
            "numFound": 1,
            "start": 0,
            "docs": [{
                "id": "978-1857995879",
                "cat": [
                    "book",
                    "paperback"
                ],
                "name": ["Sophie's World : The Greek Philosophers"],
                "author": ["Jostein Gaarder"],
                "sequence_i": 1,
                "genre_s": "fantasy",
                "inStock": [true],
                "price": [3.07],
                "pages_i": 64,
                "_version_": 1532134320281485312
            }]
        }
    }
    ```
    > Solr Start Script Reference: <https://cwiki.apache.org/confluence/display/solr/Solr+Start+Script+Reference>，有命令和参数的详细说明

## 关闭 Solr
```
bin/solr stop -all
```

## 创建 Core or Collection
```
bin/solr create -c <name>
```

## 删除 Core or Collection
```
bin/solr delete -c ebag
```

## Admin 工具
* 访问 <http://localhost:8983/solr>
* 选择 Core，例如 `ebag`
* 选择 `Query`
* 在 `q` 下面输入要搜索的内容例如 `Greek` 替换 `*:*`
* 点击 `Execute Query`，右边会列出搜索到的内容
* 或者直接用 URL 进行搜索: <http://localhost:8983/solr/ebag/select?indent=on&wt=json&q=Greek>
    * q 为搜索的内容
    * wt 为搜索结果返回的格式，默认为 xml
    * indent 为是否对结果进行缩进

## Solr 的重要概念
* Core (Collection): Document 的集合
* Document: 相当于数据库里的一个记录，Field 的集合
* Field: 数据项
* Schema: The schema is the place where you tell Solr how it should build indexes from input documents.

## 添加索引
Solr 支持多种数据格式，例如 PDF，Html，Json，Xml 等。

```
bin/post -c ebag docs/
bin/post -c ebag example/exampledocs/*.xml
bin/post -c ebag example/exampledocs/books.json
```

* ebag 是 collection 的名字
* 把目录 docs 下的所有文件都添加到 Solr 的索引里(递归添加)
* 对目录 exampledocs 中的所有 xml 做索引
* 对 books.json 做索引
* 多次对同一个文件做上面的索引操作，不会出现多个，只会更新，因为索引是以 id 来区分的(`uniqueKey`)，每个 Document 都有一个 id

## 常用搜索
* 搜索所有 document 的所有 field，包含关系
    * <http://localhost:8983/solr/ebag/select?indent=on&wt=json&q=Greek>
* 搜索指定的 field，精确匹配，`q=field:value`
    * <http://localhost:8983/solr/ebag/select?indent=on&wt=json&q=name:Greek>
* 搜索指定的 field，模糊匹配
    * <http://localhost:8983/solr/ebag/select?indent=on&wt=json&q=name:*Greek*>
* 限定要显示的 field，`fl`
    * <http://localhost:8983/solr/ebag/select?wt=json&indent=true&q=Greek&fl=id>
* 范围搜索，q=field:[start`%20TO%20`end]，价格为 0 到 10
    * <http://localhost:8983/solr/ebag/select?wt=json&indent=on&fl=id,name,price&q=price:[0%20TO%2010]>
* Facet 查询，统计 `facet.field` 属性在搜索结果中有多少个: `The facet information shows how many of the query results have each possible value of the cat field.`
    * <http://localhost:8983/solr/ebag/select?q=price:[0%20TO%20400]&facet=on&facet.field=cat&wt=json&indent=on>

## 参考
* [Running Solr](https://cwiki.apache.org/confluence/display/solr/Running+Solr)

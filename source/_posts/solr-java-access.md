---
title: Java 访问 Solr
date: 2016-04-21 14:13:07
tags: [Java, Solr]
---

可以使用 SolrJ 来访问 Solr，其提供的 SolrClient 能够在连接断开后自动重连。

<!--more-->

## Gradle 依赖
```
compile 'org.apache.solr:solr-solrj:6.0.0'
```

## 连接 Solr
```java
// ebag 是 Core 或者 Collection
SolrClient solr = new HttpSolrClient("http://localhost:8983/solr/ebag");
```

## 插入
```java
    /**
     * 创建或者更新 document:
     *      如果有存在 document 的 id 和输入的 id 相同, 则删除 document, 然后创建一个新的
     *      如果没有 document 的 id 和输入的 id 相同, 则创建
     * @throws IOException
     * @throws SolrServerException
     */
    public void insert() throws IOException, SolrServerException {
        SolrInputDocument document = new SolrInputDocument();
        document.setField("id", 10000); // 如果没有指定 id 属性, 则 Solr 会自动创建一个
        document.setField("name", "Alice");
        document.setField("age", 23);

        solr.add(document);
        solr.commit();
    }
```

## 删除
```java
    public void delete() throws IOException, SolrServerException {
        solr.deleteById("10000");
        solr.commit();
    }
```

## 查询
```java
    public void select() throws IOException, SolrServerException {
        SolrQuery query = new SolrQuery();
        query.set("wt", "json"); // 设置查询参数
        query.set("q", "name:Alice");
        QueryResponse response = solr.query(query);
        SolrDocumentList documents = response.getResults();

        System.out.println("Num Found: " + documents.getNumFound());

        for (SolrDocument document : documents) {
            System.out.println("---------------------------------------");

            Collection<String> fieldNames = document.getFieldNames();

            for (String fieldName : fieldNames) {
                System.out.println(fieldName + " : " + document.get(fieldName));
            }
        }
    }
```

## 插入 Bean
前面使用 `document.set(field, value)` 来创建 document，如果属性太多时，就容易出错，可以使用 Bean 的方式来进行插入。先定一个 Java Bean，需要插入到 Solr 的属性给其加上 `@Field` 注解，然后使用这个 Bean 创建对象 bean，调用 `solr.addBean(bean)` 插入到 Solr 里。

```java
    public void insertBean() throws IOException, SolrServerException {
        Person p1 = new Person("person-10001", "Bob", 34);
        Person p2 = new Person("person-10002", "John", 88);
        Person p3 = new Person("person-10003", "Alice", 88);

        solr.addBean(p1);
        solr.addBean(p2);
        solr.addBean(p3);
        solr.commit();
    }
```

```java
import org.apache.solr.client.solrj.beans.Field;

public class Person {
    @Field
    private String id;
    @Field
    private String name;
    @Field
    private int age;

    public Person() {}

    public Person(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
    
    // Getters and setters
}
```

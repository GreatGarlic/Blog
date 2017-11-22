---
title: Mybatis 语法
date: 2016-10-15 22:32:37
tags: SpringWeb
---

## 使用 MySQL 自动生成的主键
```xml
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO person(name, password) VALUES(#{name}, #{password})
</insert>
```

## 使用 Oracle 序列生成的主键
```xml
<insert id="insertEnrollment" parameterType="EnrollmentForm">
    <selectKey keyProperty="enrollId" resultType="long" order="BEFORE">
        SELECT S_ENR_ID.Nextval from DUAL
    </selectKey>

    INSERT INTO enrollment (id, address) VALUES (#{enrollId}, #{address})
</insert>
```

## 使用 MySQL 自动生成 UUID

```xml
<insert id="createKnowledgePoint" parameterType="KnowledgePoint">
    <selectKey keyProperty="knowledgePointId" resultType="string" order="BEFORE">
        SELECT uuid() FROM dual
    </selectKey>

    INSERT INTO knowledge_point(knowledge_point_id, name, knowledge_point_group_id, is_deleted)
    VALUES(#{knowledgePointId}, #{name}, #{knowledgePointGroupId}, 0)
</insert>
```



## 使用 LIKE 语句

```
MySql:
SELECT * FROM user WHERE name like CONCAT('%',#{name},'%')

Oracle:
SELECT * FROM user WHERE name like CONCAT('%',#{name},'%') 或 
SELECT * FROM user WHERE name like '%'||#{name}||'%'

SQLServer:  
SELECT * FROM user WHERE name like '%'+#{name}+'%'

DB2:
SELECT * FROM user WHERE name like CONCAT('%',#{name},'%') 或  
SELECT * FROM user WHERE name like '%'||#{name}||'%'
```

<!--more-->

## 使用 @Param 传递多个参数
传递多个参数可以使用

* `Map`
* `JavaBean` 中存放多个属性
* `@Param`

```java
public List<User> findUsers(@Param("offset") int offset, @Param("count") int count);
```

## #{name} 与 ${name} 的区别

`#{name}` 会根据传进来的参数的类型自动加上相应的信息，例如字符串两边会加上 `''`，日期对象会自动的转化成 SQL 识别的内容，可以`防止 SQL 注入攻击`。

`${name}` 直接替换，例如传进来的是字符串，不会在字符串两边加上 `''`，使用的场景有如 `ORDER BY`，`表名` 等

## 返回 Boolean

返回 Boolean 的 SQL 需要使用 **EXISTS**，因为 MyBatis 中 1 代表 true，非 1 代表 false，如果用 count 的话，大于 1 的情况返回 false，这是不对的。

```xml
<!--检查目录是否存在-->
<select id="isDirectoryExisting" parameterType="string" resultType="boolean">
    SELECT EXISTS(SELECT 1 FROM directory WHERE directory_id=#{directoryId})
</select>
```

## if

```xml
<!-- 查询学生 list，like 姓名 -->   
<select id="getStudentListLikeName" parameterType="StudentEntity" resultMap="studentResultMap">   
    SELECT * from STUDENT_TBL ST     
WHERE ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')    
</select>
```

但是此时如果 studentName 是 null 或空字符串，此语句很可能报错或查询结果为空。此时我们使用 `if 动态 sql 语句`先进行判断，如果值为 null 或等于空字符串，我们就不进行此条件的判断。
修改为：

```xml
<!-- 查询学生list，like姓名 -->
<select id=" getStudentListLikeName " parameterType="StudentEntity" resultMap="studentResultMap">
    SELECT * FROM STUDENT_TBL ST
    <if test="studentName!=null and studentName!='' ">
        WHERE ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')
    </if>
</select>
```

此时，当 studentName 的值为 null 或 '' 的时候，我们并不进行 where 条件的判断，所以查询结果是全部。

## where

由于参数是 Java 的实体类，所以我们可以把所有条件都附加上，使用时比较灵活， new 一个这样的实体类，我们需要限制那个条件，只需要附上相应的值就会 WHERE 这个条件，相反不去赋值就可以不在 WHERE 中判断。

`<where>` 标签会知道如果它包含的标签中有返回值的话，它就插入一个 `WHERE`。此外，如果标签返回的内容是以 `AND` 或 `OR` 开头的，则它会剔除掉。

```xml
<!-- 查询学生list，like姓名，=性别、=生日、=班级，使用where,参数entity类型 -->
<select id="getStudentListWhereEntity" parameterType="StudentEntity" resultMap="studentResultMap">
    SELECT * FROM STUDENT_TBL ST
    <where>
        <if test="studentName!=null and studentName!='' ">
            ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')
        </if>
        <if test="studentSex!= null and studentSex!= '' ">
            AND ST.STUDENT_SEX = #{studentSex}
        </if>
        <if test="studentBirthday!=null">
            AND ST.STUDENT_BIRTHDAY = #{studentBirthday}
        </if>
        <if test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' ">
            AND ST.CLASS_ID = #{classEntity.classID}
        </if>
    </where>
</select>
```

## set

当在 update 语句中使用 `<if>` 标签时，如果前面的 `<if>` 没有执行，则或导致逗号多余错误。使用 `<set>` 标签可以将动态的配置 SET 关键字，和剔除追加到条件末尾的任何不相关的逗号。
没有使用 `<if>` 标签时，如果有一个参数为 null，都会导致错误，如下示例：

```xml
<!-- 更新学生信息 -->
<update id="updateStudent" parameterType="StudentEntity">
    UPDATE STUDENT_TBL
       SET STUDENT_TBL.STUDENT_NAME = #{studentName},
           STUDENT_TBL.STUDENT_SEX = #{studentSex},
           STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday},
           STUDENT_TBL.CLASS_ID = #{classEntity.classID}
     WHERE STUDENT_TBL.STUDENT_ID = #{studentID}
</update>
```

使用 `<set>` + `<if>` 标签修改后，如果某项为 null 则不进行更新，而是保持数据库原值。如下示例：

```xml
<!-- 更新学生信息 -->
<update id="updateStudent" parameterType="StudentEntity">
    UPDATE STUDENT_TBL
    <set>
        <if test="studentName!=null and studentName!='' ">
            STUDENT_TBL.STUDENT_NAME = #{studentName},
        </if>
        <if test="studentSex!=null and studentSex!='' ">
            STUDENT_TBL.STUDENT_SEX = #{studentSex},
        </if>
        <if test="studentBirthday!=null ">
            STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday},
        </if>
        <if test="classEntity!=null and classEntity.classID!=null and classEntity.classID!='' ">
            STUDENT_TBL.CLASS_ID = #{classEntity.classID}
        </if>
    </set>
    WHERE STUDENT_TBL.STUDENT_ID = #{studentID};
</update>
```

## choose-when-otherwise

有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。MyBatis 提供了 `<choose>` 元素，按顺序判断 `<when>` 中的条件出否成立，如果有一个成立，则 `<choose>` 结束。当 `<choose>` 中所有`<when>` 的条件都不满则时，则执行 `<otherwise>` 中的 SQL。类似于 Java 的 switch 语句，choose 为 switch，when 为 case，otherwise 则为 default。

| MyBatis   | Java    |
| --------- | ------- |
| choose    | switch  |
| when      | case    |
| otherwise | default |

例如下面例子，同样把所有可以限制的条件都写上，方面使用。选择条件顺序，when标签的从上到下的书写顺序：

```xml
<!-- 查询学生list，like姓名、或=性别、或=生日、或=班级，使用choose -->
<select id="getStudentListChooseEntity" parameterType="StudentEntity" resultMap="studentResultMap">
    SELECT * from STUDENT_TBL ST
    <where>
        <choose>
            <when test="studentName!=null and studentName!='' ">
                    ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')
            </when>
            <when test="studentSex!= null and studentSex!= '' ">
                    AND ST.STUDENT_SEX = #{studentSex}
            </when>
            <when test="studentBirthday!=null">
                AND ST.STUDENT_BIRTHDAY = #{studentBirthday}
            </when>
            <when test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' ">
                AND ST.CLASS_ID = #{classEntity.classID}
            </when>
            <otherwise>

            </otherwise>
        </choose>
    </where>
</select>
```

## foreach

注意，`<foreach>` 是循环，用来读取传入的 list 参数。批量处理是 parameterType 的类型必须要注意。`<foreach>` 标签中的 `collection` 属性表示传入的是什么集合类型，`item` 表示的是集合中的一个量类似于

```java
List<String> list;
for (String str : list) {
    …… 
}
```

* `item` 就相当于 str 的作用，用来遍历 collection
* `index` 就是集合的索引
* `open` 表示标签以什么开始
* `close` 表示标签以什么结束
* `seprator` 表示元素之间的间隔

```xml
<select id="getStudentListByClassIDs" resultMap="studentResultMap">
    SELECT * FROM STUDENT_TBL ST
     WHERE ST.CLASS_ID IN
     <foreach collection="list" item="classList" open="(" close=")" separator=",">
        #{classList}
     </foreach>
</select>
```

## 批量删除
```xml
<delete id="deleteBatchByXXX" parameterType="list">
    DELETE FROM 表名 WHERE groupon_id IN
    <foreach collection="list" item="item" open="(" close =")" separator=",">
        #{item}
    </foreach >
</delete >
```

## 批量插入
```xml
<insert id="insertBatch" >
    INSERT INTO 表名 (uid, groupon_id, create_time, receive_time) VALUES
    <foreach collection="list" item="item" index ="index" separator=",">
        (#{item.uid}, #{item.grouponId}, #{item.createTime}, #{item.receiveTime})
    </foreach >
</insert>
```

## 批量更新
用法和之前的基本相同，但是需要注意传入的参数是 `map` 类型。

```java
public void batchUpdateStudentWithMap(){  
    List<Integer> ls = new ArrayList<Integer>();  
    for(int i = 2;i < 8;i++){  
        ls.add(i);  
    }  
    Map<String,Object> map = new HashMap<String,Object>();  
    map.put("idList", ls);  
    map.put("name", "mmao789");  
    SqlSession session = SessionFactoryUtil.getSqlSessionFactory().openSession();  
    session.insert("mybatisdemo.domain.Student.batchUpdateStudentWithMap",map);  
    session.commit();  
    session.close();  
}
```

```xml
<update id="batchUpdateStudentWithMap" parameterType="java.util.Map" >  
    UPDATE STUDENT SET name = #{name} WHERE id IN   
    <foreach collection="idList" index="index" item="item" open="(" separator="," close=")">   
        #{item}   
    </foreach>  
</update>
```

## 更新单条记录
```sql
UPDATE course SET name = 'course1' WHERE id = 'id1'
```

## 更新多条记录的同一个字段为同一个值
```sql
UPDATE course SET name = 'course1' WHERE id in ('id1', 'id2', 'id3)
```

## 更新多条记录为多个字段为不同的值
比较普通的写法，是通过循环，依次执行update语句

```xml
<update id="updateBatch" parameterType="java.util.List">
    <foreach collection="list" item="item" index="index" open="" close="" separator=";">
        UPDATE course
        <set>
            name=${item.name}
        </set>
        WHERE id=${item.id}
    </foreach>
</update>
```

一条记录 update 一次，性能比较差，容易造成阻塞。
MySQL没有提供直接的方法来实现批量更新，但可以使用case when语法来实现这个功能。

```sql
UPDATE course
    SET name = CASE id 
        WHEN 1 THEN 'name1'
        WHEN 2 THEN 'name2'
        WHEN 3 THEN 'name3'
    END, 
    title = CASE id 
        WHEN 1 THEN 'New Title 1'
        WHEN 2 THEN 'New Title 2'
        WHEN 3 THEN 'New Title 3'
    END
WHERE id IN (1,2,3)
```

这条 SQL 的意思是，如果 id 为 1，则 name 的值为 name1，title 的值为 New Title1；依此类推。
在Mybatis中的写法则如下：

```xml
<update id="updateBatch" parameterType="list">
    UPDATE course
    <trim prefix="SET" suffixOverrides=",">
        <trim prefix="peopleId=CASE" suffix="END,">
            <foreach collection="list" item="i" index="index">
                <if test="i.peopleId!=null">WHEN id=#{i.id} THEN #{i.peopleId}</if>
            </foreach>
        </trim>

        <trim prefix="roadgridid=CASE" suffix="END,">
            <foreach collection="list" item="i" index="index">
                <if test="i.roadgridid!=null">WHEN id=#{i.id} THEN #{i.roadgridid}</if>
            </foreach>
        </trim>

        <trim prefix="type=CASE" suffix="END," >
            <foreach collection="list" item="i" index="index">
                <if test="i.type!=null">WHEN id=#{i.id} THEN #{i.type}</if>
            </foreach>
        </trim>

        <trim prefix="unitsid=CASE" suffix="END," >
            <foreach collection="list" item="i" index="index">
                <if test="i.unitsid!=null">WHEN id=#{i.id} THEN #{i.unitsid}</if>
            </foreach>
        </trim>
    </trim>
    WHERE
    <foreach collection="list" separator="or" item="i" index="index">id=#{i.id}</foreach>
</update>
```

## 特殊字符和 CDATA

SQL 使用时特殊字符有 `<` 和 `>`，可以使用 `<![CDATA[ ]]>` 把 SQL 语句括起来:

```xml
<select id="findAreas" parameterType="int" resultType="Area"><![CDATA[
    SELECT id, name, level FROM area
    WHERE id>#{id}
]]></select>
```

> 转义也可以，但是不够漂亮。

## 一对一 association

```xml
<!-- [[3]] 使用 resultMap 映射，属性是另一个类的对象: association -->
<select id="selectFullUserById" parameterType="int" resultMap="userResultMap" >
    SELECT
        user.id      as id,
        user.age     as age,
        user.name    as name,
        ui.id        as user_info_id,
        ui.user_id   as user_info_user_id,
        ui.telephone as user_info_telephone,
        ui.address   as user_info_address
    FROM user
    ...
</select>

<resultMap id="userResultMap" type="com.xtur.bean.User" >
    <id property="id" column="id"/>
    <result property="age" column="age"/>
    <result property="name" column="name"/>
    <!--嵌套映射中还可以使用 resultMap: association, collection
    还可以使用嵌套查询，但是会产生 N＋1 问题，在大数量的数据库里会有很大的性能问题-->

    <!--<association property="userInfo" column="user_info_id" javaType="domain.UserInfo">
        <id     property="id"        column="user_info_id"/>
        <result property="userId"    column="user_info_user_id"/>
        <result property="telephone" column="user_info_telephone"/>
        <result property="address"   column="user_info_address"/>
    </association>-->

    <!--使用 columnPrefix 可以使 result map 重用-->
    <association property="userInfo" column="user_info_id" columnPrefix="user_info_" resultMap="userInfoResultMap"/>
</resultMap>
<resultMap id="userInfoResultMap" type="com.xtur.bean.UserInfo" >
    <id     property="id"        column="id"/>
    <result property="userId"    column="user_id"/>
    <result property="telephone" column="telephone"/>
    <result property="address"   column="address"/>
</resultMap>
```

## 一对多 collection

```xml
<select id="findPapersBySubjectAndNameFilter" resultMap="paperResultMap">
    SELECT 
        p.paper_id            as paper_id,
        p.name                as name,
        p.uuid_name           as uuid_ame,
        p.original_name       as original_name,
        kp.name               as kp_name,
        kp.knowledge_point_id as kp_knowledge_point_id
    FROM paper AS p
    ...
</select>

<resultMap id="paperResultMap" type="Paper">
    <id property="paperId"          column="paper_id"/>
    <result property="name"         column="name"/>
    <result property="uuidName"     column="uuid_name"/>
    <result property="originalName" column="original_name"/>

    <collection property="knowledgePoints" ofType="KnowledgePoint" column="paper_id" columnPrefix="kp_" resultMap="knowledgePointResultMap"/>
</resultMap>

<resultMap id="knowledgePointResultMap" type="KnowledgePoint">
    <id property="knowledgePointId" column="knowledge_point_id"/>
    <result property="name"         column="name"/>
</resultMap>
```

> collection 比 association 多一个 ofType。

## resultType 和 resultMap

resultType 是指已有类型，例如 int, string, class User 等，SQL 查询得到的列会自动的映射到对应的类型或者其属性上。

resultMap 是指我们自己用 XML element `<resultMap>...</resultMap>` 定义的映射。

## 参考资料

* [Mybatis之批量更新操作](http://my.oschina.net/ckanner/blog/338515)
* [Mybatis 使用经验分享之批量操作](http://jingyan.baidu.com/article/11c17a2c7f376af446e39d21.html)
* [Mybatis 的 where foreach set 等标签详解](http://blog.csdn.net/zenson_g/article/details/10137665)



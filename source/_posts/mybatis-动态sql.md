---
title: 动态sql
categories: mybatis
tags:
  - mybatis
excerpt: ''
cover: https://b.zol-img.com.cn/desk/bizhi/image/10/960x600/1604539876877.jpg
---

#### 动态sql常用组合

- where...if...
- choose...when...otherwise...
- set...if
- trim
- bind
- foreach
- include

#### where...if...

> where元素只会在子元素返回内容的情况下才插入 "where"子句。而且，若子句的开头为 "and"或 "or"，where元素也会将他们自动去除

~~~xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from blog
    <where>
        <if test="title != null">
            title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </where>
</select>
~~~

#### trim

~~~xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
</trim>
~~~

#### set...if...

> set 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号

~~~xml
<!--注意set是用的逗号隔开-->
<update id="updateBlog" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
    where id = #{id};
</update>
~~~

#### choose...when...otherwise

> 不想用到所有的查询条件，只想选择其中的一个，查询条件有一个满足即可，使用 choose 标签可以解决此类问题，类似于 Java 的 switch 语句

~~~xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
    select * from blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
~~~

#### foreach

~~~xml
<select id="queryBlogForeach" parameterType="map" resultType="blog">
    select * from blog
    <where>
  <!--
  collection:指定输入对象中的集合属性
  item:每次遍历生成的对象
  open:开始遍历时的拼接字符串
  close:结束时拼接的字符串
  separator:遍历对象之间需要拼接的字符串
  select * from blog where 1=1 and (id=1 or id=2 or id=3)
  -->
        <foreach collection="ids" item="id" open="and (" close=")"
                 separator="or">
            id=#{id}
        </foreach>
    </where>
</select>
~~~

#### include

~~~xml
//最好基于 单表来定义 sql 片段，提高片段的可重用性
//在 sql 片段中不要包括 where
<sql id="if-title-author">
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</sql>

<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from blog
    <where>
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace-->
        <include refid="if-title-author"></include>
        <!-- 在这里还可以引用其他的 sql 片段 -->
    </where>
</select>
~~~

#### bind

> bind 元素允许你在 OGNL 表达式以外创建一个变量，并将其绑定到当前的上下文。

~~~xml
<select id="selectBlogsLike" resultType="blog">
    <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
    SELECT * FROM BLOG
    WHERE title LIKE #{pattern}
</select>
~~~

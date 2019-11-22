---
title: Mybatis-resultMap
date: 2018-12-07 15:54:08
tags: [Mybatis]
categories: Mybatis
---

我们可以这样来写一个select statement：

```
<select id="selectById" parameterType="int" resultType="com.user.pojo.UserPojo">
  select id,name,create_time from user where id= #{id}
</select>
```

<!-- more -->

用于接收查询结果的resultType为UserPojo，即：

```
package com.user.pojo;

import java.util.Date;

public class UserPojo {

    private int id;

    private String name;

    private Date create_time;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getCreate_time() {
        return create_time;
    }

    public void setCreate_time(Date create_time) {
        this.create_time = create_time;
    }
}
```

即要求UserPojo中必须有同名的id,name,create_time(类型自然也要能匹配，getter及setter方法倒不要求)，否则，例如我们将UserPojo中的create_time改名为createTime，此时就无法填入该值。

但是，这委实又是一个常见的需求：Mysql中的字段名通常使用下划线，而Java的字段名则更倾向于使用驼峰式。因为这种命名规范上的不一致而改变一方(正如上文中Java代码中的create_time字段)显然是很奇怪的。

解决方式之一是在sql中下功夫：

```
SELECT id,name,create_time AS createTime FROM user WHERE id= #{id}
```

而在不方便修改如此sql的场合，则可以使用resultMap，它的作用是完成sql查询出的列名与pojo中字段名的映射。

例如，我们可以这样来写mapper配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.user.mapper.UserMapper">
  <!-- id:唯一标识一个resultMap -->
  <!-- type:最终要映射到的pojo的类型 -->
  <resultMap id="userMap" type="UserPojo">
    <!-- id是主键列(本例其实是不需要映射的，仍然写出id标签只是为了举例说明)，result是非主键列 -->
    <!-- column:sql语句查询出的列名 -->
    <!-- type:pojo类型中的字段名 -->
    <id column="id" property="id" />
    <result column="create_time" property="createTime" />
  </resultMap>
  <select id="selectById" parameterType="int" resultMap="userMap">
    SELECT id,name,create_time AS createTime FROM user WHERE id= #{id}
  </select>
  <select id="selectByName" parameterType="String" resultMap="userMap">
    select id,name,create_time AS createTime from user where name like '${value}%'
  </select>
  <insert id="insert" parameterType="UserPojo">
    <selectKey keyProperty="id" order="AFTER" resultType="int">
      select last_insert_id()
    </selectKey>
    insert into user(name,create_time) values(#{name},#{createTime})
  </insert>
  <delete id="delete" parameterType="int">
    delete from user where id=#{id}
  </delete>
  <update id="update" parameterType="UserPojo">
    update user set name=#{name},create_time=#{createTime} where id=#{id}
  </update>
</mapper>
```

mapper标签下新增resultMap标签。
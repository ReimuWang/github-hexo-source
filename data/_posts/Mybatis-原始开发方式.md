---
title: Mybatis-原始开发方式
date: 2018-10-04 11:27:08
tags: [Mybatis]
categories: Mybatis
---

# 引入依赖

在此只给出欲使用Mybatis的依赖的最小集：

```
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.43</version>
</dependency>
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.4.4</version>
</dependency>
<dependency>
  <groupId>asm</groupId>
  <artifactId>asm</artifactId>
  <version>3.3.1</version>
</dependency>
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.2.5</version>
</dependency>
<dependency>
  <groupId>org.javassist</groupId>
  <artifactId>javassist</artifactId>
  <version>3.21.0-GA</version>
</dependency>
```

<!-- more -->

# 全局配置文件

遵循[Mybatis-概述](/2018/10/04/Mybatis-概述/)的建议，我们将全局配置文件命名为SqlMapConfig.xml：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 环境配置，与Spring整合后，环境配置将交由Spring管理，彼时environments标签废弃 -->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="" />
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="mybatis/mapper/UserMapper.xml" />
  </mappers>
</configuration>
```

# mapper配置文件

遵循[Mybatis-概述](2018/10/04/Mybatis-概述/)的建议，我们将mapper配置文件命名为UserMapper.xml(本示例只需操作一张表，因此mapper配置文件仅有一个)：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace为命名空间，其作用为分类管理SQL语句 -->
<!-- 只要不重名，namespace可随意命名。不过使用mapper代理开发方式后，namespace必须与Mapper接口同名 -->
<mapper namespace="loveReimu">
  <!-- namespace+id唯一标识一个statement -->
  <select id="selectById" parameterType="int" resultType="com.user.pojo.UserPojo">
    select * from user where id= #{id}
  </select>
  <select id="selectByName" parameterType="String" resultType="com.user.pojo.UserPojo">
    select * from user where name like '${value}%'
  </select>
  <insert id="insert" parameterType="com.user.pojo.UserPojo">
    <selectKey keyProperty="id" order="AFTER" resultType="int">
      select last_insert_id()
    </selectKey>
    insert into user(name,create_time) values(#{name},#{create_time})
  </insert>
  <delete id="delete" parameterType="int">
    delete from user where id=#{id}
  </delete>
  <update id="update" parameterType="com.user.pojo.UserPojo">
    update user set name=#{name},create_time=#{create_time} where id=#{id}
  </update>
</mapper>
```

正常写SQL脚本时可在SQL的末尾加上分号以分割。Mybatis不支持SQl后加分号。

---

基本单位为statement，即一个select/insert/delete/update。可以认为每个statement都封装了一条SQL语句。其内部会通过parameterType完成输入映射；通过resultType完成输出映射。

---

#{}表示一个占位符。它会自动进行Java类型向JDBC类型值的转换。例如传入一个字符串，实际执行时会自动在两边加单引号。再比如传入一个字符串类型的参数至数据库中的datetime类型，mysql会自动将其转换为日期类型(Oricle不支持该功能，传入时就必须传入日期类型)。

${}表示SQL的拼接符。它类似于#{}，不同之处在于不会自动进行Java类型向JDBC类型值的转换，即实际执行时维持原值。此时PreparedStatement的防注入功能失效。

---

#{}或${}接收pojo类型的parameterType时，会使用OGNL以对象导航图的方式解析pojo的属性值。简单来说，就是可以一直.出所需参数值。例如parameterType传入的pojo类型为User，User中的sex属性同样为pojo类型。则若要取到sex中的real字段，则可用#{sex.real}。

OGNL在解析对象时不需要对应字段提供getter，setter方法。

---

parameterType为传入参数类型，规定只能传入一个参数(因此如果想传入多个值的话，只能通过pojo的OGNL来保证了)。

```
parameterType="String"
```

也可写为：

```
parameterType="string"
```

或：

```
parameterType="java.lang.String"
```

如果parameterType为hashmap，则使用方式与使用OGNL解析pojo时类似。

---

resultType为输出结果类型：

- pojo
- hashmap: 会将 字段名-字段值 映射为hashmap的 key-value
- Java基本类型: 要求查询的结果必须仅有1行1列，且结果值与对应基本类型能互相转换

无论查询结果返回的是一条或多条记录，resultType均指定单条记录映射的类型。若返回的是多条记录，Mybatis会自动将其封装为List。

---

除parameterType外，还提供parameterMap配置输入参数类型(已过时，不推荐使用)。

除resultType外，还提供resultMap配置输出结果类型(未过时，仍推荐使用)完成复杂数据类型映射(例如一对多映射，多对多映射)

---

本例中，insert标签中的selectKey类似于habernate的主键返回功能。作用为得到新插入数据自增的主键值(selectKey并不仅仅可以得到主键，这只是它的应用之一)。

Mysql中的last_insert_id()函数可获得刚插入的自增主键值。

selectKey标签中的属性：

- keyProperty：将查得的key赋给的属性名。
- order：返回时机。AFTER即为在本条SQL执行完成后得到所需key值。
- resultType：selectKey标签中的SQL语句的返回值类型。

若所用数据库为Orical，因Orical没有自增主键查询函数，则需使用序列实现主键生成。

---

若Mysql表中主键未设置自增，则可用uuid函数生成主键。此时生成的时机应为本条语句执行之前。并将生成的主键作为参数插入数据库中。
使用uuid函数的好处为可保证主键在数据库表合并时始终全局唯一。

此时的insert statement应这样写：

```
<insert id="insert" parameterType="com.day1_5.user.pojo.UserPojo">
  <selectKey keyProperty="id" order="BEFORE" resultType="string">
    select uuid()
  </selectKey>
  insert into user2(id,name,create_time) values(#{id},#{name},#{create_time})
</insert>
```

注意：Mysql uuid()生成的id长度为36位，因此用于接收的varchar字段长度至少要为36。最后得到的id形如：

```
6f644312-3de7-11e7-910e-1002b501dff5
```

# 测试类

```
package com;

import java.util.Date;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import com.alibaba.fastjson.JSON;
import com.user.pojo.UserPojo;

public class MybatisTest {

    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws Exception {
        this.sqlSessionFactory = new SqlSessionFactoryBuilder()
	                         .build(Resources.getResourceAsStream("mybatis/SqlMapConfig.xml"));
    }

    @Test
    public void selectById() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserPojo userPojo = null;
        try {
            userPojo = sqlSession.selectOne("loveReimu.selectById", 2);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sqlSession.close();
        }
        System.out.println(JSON.toJSONString(userPojo));
    }

    @Test
    public void selectByName() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<UserPojo> userPojoList = null;
        try {
            userPojoList = sqlSession.selectList("loveReimu.selectByName", "八云");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sqlSession.close();
        }
        System.out.println(JSON.toJSONString(userPojoList));
    }

    @Test
    public void insert() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserPojo userPojo = new UserPojo();
        userPojo.setName("八意永琳");
        userPojo.setCreate_time(new Date());
        try {
            // 该方法返回插入操作影响的记录数
            sqlSession.insert("loveReimu.insert", userPojo);
            sqlSession.commit();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sqlSession.close();
        }
        // 因配置文件中设置了selectKey，userPojo中会填入新生成的自增主键值
        System.out.println(JSON.toJSONString(userPojo));
    }

    @Test
    public void delete() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
            System.out.println(sqlSession.delete("loveReimu.delete", 1));
            sqlSession.commit();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sqlSession.close();
        }
    }

    @Test
    public void update() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserPojo userPojo = new UserPojo();
        userPojo.setId(4);
        userPojo.setName("蓬莱山辉夜");
        userPojo.setCreate_time(new Date());
        try {
            System.out.println(sqlSession.update("loveReimu.update", userPojo));
            sqlSession.commit();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sqlSession.close();
        }
    }
}
```

我们使用SqlSession操作数据库，它的生成过程为：

```
SqlSessionFactoryBuilder - SqlSessionFactory - SqlSession
```

其中SqlSessionFactoryBuilder与SqlSessionFactory通常均是单例的，而SqlSession则应与查询一一对应。这是为什么呢？我们不妨看一下SqlSession的类定义：

```
package org.apache.ibatis.session;

public interface SqlSession extends Closeable
```

它有两个实现类：

```
package org.apache.ibatis.session.defaults;

public class DefaultSqlSession implements SqlSession
```

```
package org.apache.ibatis.session;

public class SqlSessionManager implements SqlSessionFactory, SqlSession
```

其中DefaultSqlSession为默认的实现类，在该类中，有一个成员变量为：

```
private Executor executor;
```

即为：

```
package org.apache.ibatis.executor;

public interface Executor
```

SqlSession内部会调用Excutor接口操作数据库。它有两个实现：

默认执行器：

```
package org.apache.ibatis.executor;

public abstract class BaseExecutor implements Executor
```

缓存执行器：

```
package org.apache.ibatis.executor;

public class CachingExecutor implements Executor
```

以默认执行器的某个子类(批量操作执行器)为例：

```
package org.apache.ibatis.executor;

public class BatchExecutor extends BaseExecutor
```

BatchExecutor有如下成员变量：

```
private String currentSql;    // 封装SQL语句
private MappedStatement currentStatement;    // 用于封装SQL的对象。其中有statement中定义的所有内容。
```

MappedStatement类成员变量举例：

```
private String resource;
private Configuration configuration;
private String id;
private Integer fetchSize;
private Integer timeout;
private StatementType statementType;
private ResultSetType resultSetType;
private SqlSource sqlSource;
private Cache cache;
private ParameterMap parameterMap;
private List<ResultMap> resultMaps;
private boolean flushCacheRequired;
private boolean useCache;
private boolean resultOrdered;
private SqlCommandType sqlCommandType;
private KeyGenerator keyGenerator;
private String[] keyProperties;
private String[] keyColumns;
private boolean hasNestedResultMaps;
private String databaseId;
private Log statementLog;
private LanguageDriver lang;
private String[] resultSets;
```

显然，SqlSession是线程不安全的。诸如currentSql这样的属性必须要与查询一一对应。

---

selectById()方法在查询单条纪录时，调用了sqlSession.selectOne()方法：

```
<T> T selectOne(String statement, Object parameter);
```

selectByName()方法在查询多条纪录时，调用了sqlSession.selectList()方法：

```
<E> List<E> selectList(String statement, Object parameter);
```

这两个方法的第一个参数statement，均是mapper配置文件中namespace+id构成的唯一值。

selectOne()不能用于查询复数结果，会报错；反之，selectList()则可用于查询单条记录(可以认为list中只有一条记录)。

---

与Spring整合后，诸如：

```
sqlSession.commit();
```

或

```
sqlSession.close();
```

均可交由Spring管理。
---
title: Mybatis-mapper代理开发方式
date: 2018-10-04 13:40:08
tags: [Mybatis]
categories: Mybatis
---

在[Mybatis-DAO开发方式](/2018/10/04/Mybatis-DAO开发方式/)的基础上，我们引入mybatis特有的，也是推荐大家使用的mapper代理开发方式，此时程序员仅需编写mapper接口，mybatis将自动生成mapper实现的代理对象。

<!-- more -->

那么mybatis要基于什么规则来生成mapper实现的代理对象呢？而它的前置问题则是，mapper实现究竟是什么样子的呢？

我们知道，所谓mapper接口其实就是DAO接口，而mapper实现则是DAO实现。以[Mybatis-DAO开发方式](/2018/10/04/Mybatis-DAO开发方式/)为例，它是这样的：

```
package com.user.dao;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

import com.user.pojo.UserPojo;

public class UserDaoImpl implements UserDao {

    private SqlSessionFactory sqlSessionFactory;

    public UserDaoImpl(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    @Override
    public UserPojo selectById(int id) {
        SqlSession sqlSession = null;
        UserPojo userPojo = null;
        try {
            sqlSession = this.sqlSessionFactory.openSession();
            userPojo = sqlSession.selectOne("loveReimu.selectById", id);
        } finally {
            if (null != sqlSession) sqlSession.close(); 
        }
        return userPojo;
    }
}
```

如果想要通过代码自动生成一段这样的代码，需要准备些什么呢？

这段代码的核心其实只有1句，也就是上例中的：

```
userPojo = sqlSession.selectOne("loveReimu.selectById", id);
```

如果要让自动生成的代码实现与这句话相同的功能，那么必须做到以下两点：

1. 定位到某个精确的statement，即能定位到某个精确的namespace+id，在上例中，该值为loveReimu.selectById。
2. 明确该调用SqlSession的哪个方法，在上例中，方法为selectOne。

为做到以上两点，mybatis制定了相应的mapper开发规范。
 
在[Mybatis-DAO开发方式](/2018/10/04/Mybatis-DAO开发方式/)中，UserMapper.xml的namespace是这样的：

```
<mapper namespace="loveReimu">
```

这是一个随机值。而在mapper开发规范中，namespace必须为mapper接口的全限定名。我们不妨先定义UserMapper.xml的mapper接口：

```
package com.user.mapper;

public interface UserMapper {
}
```

接口名不是强制的，不过通常我们都会将其命名为表名+Mapper。

那么UserMapper.xml的namespace就要这样写：

```
<mapper namespace="com.user.mapper.UserMapper">
```

很显然，这是为了实现目的1。只要是UserMapper接口的方法，mybatis就会去UserMapper.xml找对应的statement。而具体该找哪一个statement的id则由mapper接口中的方法名限定。换句话说，statement的id应与mapper接口的方法名相同。例如，如果我们最终要调用UserMapper.xml中的selectById：

```
<select id="selectById" parameterType="int" resultType="com.user.pojo.UserPojo">
  select * from user where id= #{id}
</select>
```

那么mapper接口中对应的方法就要这样写：

```
UserPojo selectById(int id);
```

其中，方法的入参是statement的parameterType(很显然，因为parameterType只能有一个，方法的入参也只能有1个了)，而返回值则是statement的resultType。

这样便达成了目标1，同时也顺便达成了目标2：我们可以通过statement中的sql语句来确定该调用SqlSession的哪个方法。特别的，对于查询操作而言，方法的返回值决定了调用SqlSession的哪个查询方法：如果返回值是单个的，也就是上例中的那样，则调用selectOne。如果返回值是列表，比如说，写作：

```
List<UserPojo>
```

则调用selectList。如果明明查询返回的是list，而返回值硬要用单个UserPojo接的话，依然会遵循方法的返回值调用selectOne，同时抛出异常：

```
org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 3
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:81)
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:82)
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:59)
	at com.sun.proxy.$Proxy5.selectByName(Unknown Source)
	at com.mybatisTest.MybatisTest.selectByName(MybatisTest.java:44)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
	at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
	at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
```

随后，我们就可以写调用代码了：

```
package com.mybatisTest;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import com.alibaba.fastjson.JSON;
import com.user.mapper.UserMapper;
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
            UserMapper userMapperImpl = sqlSession.getMapper(UserMapper.class);
            userPojo = userMapperImpl.selectById(1);
        } finally {
            sqlSession.close();
        }
        System.out.println(JSON.toJSONString(userPojo));
    }
}
```

再举一个插入的例子。UserMapper.xml中的statement为：

```
<insert id="insert" parameterType="com.user.pojo.UserPojo">
  <selectKey keyProperty="id" order="AFTER" resultType="int">
    select last_insert_id()
  </selectKey>
  insert into user(name,create_time) values(#{name},#{create_time})
</insert>
```

UserMapper接口中添加对应方法：

```
int insert(UserPojo userPojo);
```

最后是调用方法：

```
@Test
public void insert() {
    UserPojo userPojo = new UserPojo();
    userPojo.setName("八意永琳");
    userPojo.setCreate_time(new Date());
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
        UserMapper userMapperImpl = sqlSession.getMapper(UserMapper.class);
        System.out.println(userMapperImpl.insert(userPojo));
        sqlSession.commit();
    } finally {
        sqlSession.close();
    }
    System.out.println(JSON.toJSONString(userPojo));
}
```

需要注意的是，

```
sqlSession.commit();
```

还是需要执行。
---
title: Mybatis-DAO开发方式
date: 2018-10-04 12:58:08
tags: [Mybatis]
categories: Mybatis
---

在[Mybatis-原始开发方式](/2018/10/04/Mybatis-原始开发方式/)的基础上，我们引入通用的DAO(Data Access Object)开发方式，此时DAO接口及DAO实现均需程序员编写。

<!-- more -->

# DAO接口

```
package com.user.dao;

import com.user.pojo.UserPojo;

public interface UserDao {
    UserPojo selectById(int id) throws Exception;
}
```

# DAO实现

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

# 测试类

```
package com;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import com.alibaba.fastjson.JSON;
import com.user.dao.UserDaoImpl;

public class MybatisTest {

    private SqlSessionFactory sqlSessionFactory;

    @Before
    public void init() throws Exception {
        this.sqlSessionFactory = new SqlSessionFactoryBuilder()
	                         .build(Resources.getResourceAsStream("mybatis/SqlMapConfig.xml"));
    }

    @Test
    public void selectById() {
        System.out.println(JSON.toJSONString(new UserDaoImpl(this.sqlSessionFactory).selectById(2)));
    }
}
```
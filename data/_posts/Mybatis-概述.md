---
title: Mybatis-概述
date: 2018-10-04 10:52:08
tags: [Mybatis]
categories: Mybatis
---
# Mybatis简介

Mybatis因其较为灵活而流行，它的前身是Apache的开源项目ibatis。2010年ibatis被apache software foundation迁移至google code，改进后更名为Mybatis，目前Mybatis托管于github上。

Mybatis是一个持久层框架。使用Mybatis时，程序员只需要关注SQL本身而不需要关注使用JDBC时需进行的繁琐设置，Mybatis会将SQL很好的维护起来。它通过XML或注解的方式将要执行的各种statement(Statement、PreparedStatemnt、CallableStatement)配置起来，并通过Java对象和statement中的SQL进行映射生成最终执行的SQL语句，最后由Mybatis框架执行SQL并将结果映射成Java对象并返回。

无论如何，Java底层均使用JDBC操作关系型数据库，而Mybatis是对JDBC的封装，因此性能较之JDBC有所下降。

<!-- more -->

# Mybatis与Hibernate的区别

区别主要用于项目技术选型。进行技术选型时，主要考虑以下两点：

- 降低开发成本
- 提高系统稳定性

**Mybatis**

- 入门简单。程序容易上手开发，节省开发成本。
- 需要程序员自己编写SQL语句，是一个不完全的ORM框架(Object Relational Mapping，对象关系映射)，易于进行SQL的修改及优化。
- 适合开发需求变更频繁的项目。例如：互联网电商网站等互联网项目(讲究敏捷开发，高效)。

**Hibernate**

- 入门门槛较高。难以写出性能较高的程序(需要用到缓存技术，而且这些缓存技术已过时，现在讲究的是分布式缓存)。
- 无需写SQL语句，提倡面向对象，采用完全的标准ORM框架，无法优化SQL语句。若想优化SQL语句，则必须使用Hibernate写原生态SQL的方法，此时Hibernate的优势将不复存在，换句话说，此时已没有使用Hibernate的必要，可直接使用JDBC。
- 适合开发需求变更不大，对象数据模型稳定，中小型的项目。例如：企业OA(办公自动化，Office Automation)。

# Mybatis架构

按照逐渐远离用户(程序员)的顺序，Mybatis的架构为：

**配置文件**

- 1个全局配置文件：通常命名为SqlMapConfig.xml
- 复数个mapper.xml文件：配置具体查询某张表时的SQL。最初ibatis的命名规则为表名.xml。Mybatis不会限定命名，但建议命名为：表名+mapper.xml。这是为了在引入mapper代理开发方式后(通常都是会引入的)，保持与mapper接口同名。

**SqlSessionFactory**
 
创建SqlSession的会话工厂。

**SqlSession**

SqlSession是面向用户的接口(更直白的说，再往下的部分用户就不可见了)，接口中封装了操作数据库的方法。

**Excutor**

即为操作数据库的执行器接口，SqlSession内部调用Excutor操作数据库。Excutor接口有两个实现：

- 默认执行器
- 缓存执行器

**MappedStatement**

MappedStatement是Mybatis的底层封装对象，该对象封装了SQL语句。Excutor通过MappedStatement操作数据库。具体来说，MappedStatement接收输入映射传入的参数并将其封装为对象，而后Excutor调用该对象操作数据库，操作结束后通过输出映射生成结果对象。
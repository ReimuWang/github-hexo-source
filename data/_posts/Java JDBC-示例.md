---
title: Java JDBC-示例
date: 2017-10-12 19:31:36
tags: [Java,JDBC]
categories: Java JDBC
---

引入依赖：

```
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.43</version>
</dependency>
```

<!-- more -->

示例代码：

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class Main {

    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8", "root", "");
	    // ?表示占位符
            preparedStatement = connection.prepareStatement("select * from user where name=?");
	    // 向占位符中填入值。需要注意的是，该值从1开始
            preparedStatement.setString(1, "博丽灵梦");
            resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) System.out.println(resultSet.getString("id") + "    " + resultSet.getString("name"));
        } catch(Exception e) {
            e.printStackTrace();
        } finally {
            if (null != resultSet) {
                try {
                    resultSet.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (null != preparedStatement) {
                try {
                    preparedStatement.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (null != connection) {
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

该代码在import时，有以下冲突：

- com.mysql.cj.jdbc.PreparedStatement(C)
- java.sql.PreparedStatement(I，导入)

---

执行insert,update,delete等操作时需sqlSession.commit()，select时则无需这么做。

---

JDBC将封装SQL的单元称为statement，PreparedStatement是其中的一种，即预编译的Statement。

较之普通的statement，PreparedStatement有如下两点优势：

**防止SQL注入**

SQL注入简单举例:

SQL模版如下：

```
String sql = "select * from user where name='" + name + "'";
```

正常情况下，若传入name如下：

```
String name = "博丽灵梦";
```

则模版实际执行时的sql为：

```
select * from user where name='博丽灵梦'
```

若传入name变为：

```
String name = "博丽灵梦' or '1=1";
```

则模版实际执行时的SQL为：

```
select * from user where name='博丽灵梦' or '1=1'
```

会将表中所有数据均查询出来，形成SQL注入。

**效率高**

当使用statement时，数据库接收到SQL语句后会找寻最优执行途径即进行编译，编译的结果会记录到缓存中，下次再遇到相同的SQL则会直接读取缓存中已有的方案而跳过编译的步骤。但是在使用statement时SQL1:

```
select * from user where id=1
```

与SQL2:

```
select * from user where id=2
```

是两条不同的SQL，因此即便数据库缓存中已有SQL1的处理结果，当其接到SQL2的请求时仍需重新编译，即便这两个SQL执行的最优途径实际上是相同的。PreparedStatement使用

```
select * from user where id=?
```

规避了这个问题。
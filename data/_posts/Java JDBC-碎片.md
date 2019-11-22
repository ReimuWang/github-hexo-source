---
title: Java JDBC-碎片
date: 2017-10-12 18:31:36
tags: [Java,JDBC]
categories: Java JDBC
---

# JDBC能否处理Blob和Clob？

<!-- more -->

Blob是为存储大的二进制数据而设计的二进制大对象(Binary Large Object)。Clob是为存储大的文本数据而设计的大字符对象(Character Large Object)。JDBC的PreparedStatement和ResultSet都提供了相应的方法来支持Blob和Clob操作。

例如，表设计如下：

```
create table person_photo
(
id int primary key auto_increment,
name varchar(20) unique not null,
photo longblob
);
```

程序如下：

```
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Test {
    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement ps = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8", "root", "");
            ps = connection.prepareStatement("insert into person_photo values (default, ?, ?)");
            ps.setString(1, "八云紫");
            try (InputStream in = new FileInputStream("d:\\0.jpg")) {
                ps.setBinaryStream(2, in);
                System.out.println(ps.executeUpdate() == 1 ? "插入成功" : "插入失败");
            } catch(IOException e) {
                e.printStackTrace();
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        } finally {
            if (null != ps) {
                try {
                    ps.close();
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

输出：

```
插入成功
```

数据库受影响变为：

![0.jpg](/images/blog_pic/Java JDBC/碎片/0.jpg)
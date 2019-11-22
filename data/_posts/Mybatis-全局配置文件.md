---
title: Mybatis-全局配置文件
date: 2018-10-04 13:41:08
tags: [Mybatis]
categories: Mybatis
---

要求不高的话，SqlMapConfig.xml可以是这样的：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
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

<!-- more -->

事实上，这只是configuration标签中的很少的一部分。如果按顺序列出的话，configuration中可以配置如下标签：

- properties(属性)
- settings(全局配置参数)
- typeAliases(类型别名)
- typeHandlers(类型处理器)
- objectFactory(对象工厂)
- objectWrapperFactory
- reflectorFactory
- plugins(插件)
- environments(环境集合属性对象)
- databaseIdProvider
- mappers(映射器)

每个标签又可包含子标签，例如environments中有environment(环境子属性对象)，而environment中又包含transactionManager(事务管理)及dataSource(数据源)。

通常来说，配置文件中的标签是无序的，不过Mybatis的DTD特地约束了顺序。如果我们违背这个顺序，例如我们这样写上文的SqlMapConfig.xml：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <mappers>
    <mapper resource="mybatis/mapper/UserMapper.xml" />
  </mappers>
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
</configuration>
```

此时就会报错：

```
The content of element type "configuration" must match 
 "(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".
```

下面我们就来逐个介绍这些标签。

# properties

properties标签中有两个属性：resource及url。通常我们都会使用前者，它的作用是加载classpath下的属性文件(url通常用于加载位于网络中的属性文件)。

我们再来看下上文的SqlMapConfig.xml：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <mappers>
    <mapper resource="mybatis/mapper/UserMapper.xml" />
  </mappers>
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
</configuration>
```

此时，我们把dataSource的连接字段直接写在了SqlMapConfig.xml中。显然，在需要替换数据源的场合，这很不方便。因此我们就可以将dataSource提取为单独的properties文件，而后再由properties标签加载，这样就完成了主配置文件与配置细节的解耦。其他需灵活配置的地方也同理。例如，我们创建了properties文件db.properties:

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8
jdbc.username=root
jdbc.password=
```

此时，SqlMapConfig.xml就可以这样写了：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="mybatis/db.properties"></properties>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="mybatis/mapper/UserMapper.xml" />
  </mappers>
</configuration>
```

对于dataSource这种相对完整的数据集合而言，最好还是要单独提取出properties文件的。不过，如果属性相对简单，换句话说，不值得单独提取为文件，也可以直接写在properties标签内部：

```
<!DOCTYPE configuration PUBLIC
  "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="mybatis/db.properties">
    <property name="mapperpath.user" value="mybatis/mapper/UserMapper.xml"/>
  </properties>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="${mapperpath.user}" />
  </mappers>
</configuration>
```

如上例所示，我们将UserMapper.xml的路径放到了属性中。这样来看，properties可以理解为SqlMapConfig.xml的常量配置仓库。将可能会发生变化的常量集中在properties属性中显然是一个很好的习惯。

下面是一个老生常谈的问题，properties标签内部的key与resource/url中加载的key重名了怎么办？

Mybatis是这样处理的：

1. 首先读取properties标签内部的key
2. 而后读取resource/url中加载的key，它会覆盖已有同名属性
3. 最后，若mapper.xml配置文件中statement中的parameterType是pojo类型，那么它内部的值也会被读入，并覆盖同名属性

不得不说，这其实是挺坑爹的。因为从逻辑上来说，前两项属于系统配置，而第三项是具体的一次SQL查询，二者是不应放在一起的。不过，因为3会覆盖前两项，所以即便重名，每次SQL查询还是优先会取3中的值，因此通常不会有什么问题。

不过，对于这种放在一起的做法，我们还是很容易就会想到一些问题，假如SqlMapConfig.xml中有属性名为a，而某次statement的parameterType是pojo，且其中同样有名为a的属性。在a不为null时没什么好说的，那么如果该pojo中的a为null呢？会传入SqlMapConfig.xml中的属性a吗？

答案是不会。传入的依然会是null。

进一步的，如果pojo中没有属性a，而statement中的SQL误写了属性a，那么此时会传入SqlMapConfig.xml中的a吗？

答案依然是否定的，此时会报错，告知pojo中没有a。

因此，虽然这种将配置参数与请求参数混合的做法很坑爹，但是Mybatis还是做了相应的容错的，大家依然可以放心使用。不过为了逻辑上的顺畅，还是建议大家在定义properties用到的key时，起名特殊一些，最好一眼就能看出是配置文件中的参数。

# settings

如果我们将使用Mybatis的程序看作一个普通的软件的话，那么settings就是在配置该软件全局的运行参数。如果不加设定，会取用某个默认值。

Mybatis的前身，ibatis的settings中包含了很多性能参数(最大线程数，最长等待时间等)，而Mybatis的settings中移除了这些参数，相关性能调优会由Mybatis自动完成。

# typeAliases

有的时候mapper配置文件中parameterType/resultType指定的类型会很长(例如com.day1_9.user.pojo.UserPojo)，反复书写的话会很不方便。而且同时在多个statement中硬编码也不利于类型全限定名变更时的维护。此时就需要别名登场啦。

Mybatis定义了很多默认的别名(别名:映射的Java类型)：

- _byte: byte
- _long: long
- _short: short
- _int: int
- _integer: int
- _double: double
- _float: float
- _boolean: boolean
- string: String
- byte: Byte
- long: Long
- short: Short
- int: Integer
- integer: Integer
- double: Double
- float: Float
- boolean: Boolean
- date: Date
- decimal: BigDecimal
- bigdecimal: BigDecimal

**定义单个别名**

为了能够给用户自定义的类型起别名，可在SqlMapConfig.xml中添加typeAliases标签：

```
<typeAliases>
  <typeAlias type="com.user.pojo.UserPojo" alias="user"/>
</typeAliases>
```

在typeAliases标签内部，每个typeAlias标签都对应一个别名。其中type为类型全限定名，alias为别名。这样定义后，UserMapper.xml中的statement就可以这样写啦：

```
<select id="selectById" parameterType="int" resultType="user">
  select * from user where id= #{id}
</select>
<update id="update" parameterType="user">
  update user set name=#{name},create_time=#{create_time} where id=#{id}
</update>
```

这样，parameterType及resultType中的"com.user.pojo.UserPojo"就均被"user"替换了。

**批量别名定义**

如果pojo类有很多，那么像上文那样一个个定义就显得很麻烦了。因此Mybatis还提供了批量定义别名的方法：提供一个包，程序会自动扫描包下的Java类，它们的别名就是类名(首字母大写或小写均可)。即修改SqlMapConfig.xml的typeAliases标签如下：

```
<typeAliases>
  <package name="com.user.pojo"/>
</typeAliases>
```

如需添加复数个包，则配置对应个数个package标签即可。

批量添加别名后，UserMapper.xml中的statement就可以这样写:

```
<select id="selectById" parameterType="int" resultType="UserPojo">
  select * from user where id= #{id}
</select>
<update id="update" parameterType="UserPojo">
  update user set name=#{name},create_time=#{create_time} where id=#{id}
</update>
```

遗憾的是，Mybaits的批量扫描别名不支持通配符，也就是说如果SqlMapConfig.xml想写成这样：

```
<typeAliases>
  <package name="com.*.pojo"/>
</typeAliases>
```

是不可以的。

较之逐个配置别名，批量扫描别名的好处在于配置简单，缺点在于无法灵活命名(固定为类名)，这就会导致别名不稳定：当pojo的类名发生变动时(虽然通常这是不会变的)，还是需要修改mapper配置文件中出现的全部该pojo的别名。

# typeHandlers

typeHandlers(类型处理器)负责完成Java类型与JDBC类型之间的映射。默认提供几乎全部的常见基本类型间的映射：

![0.jpg](/images/blog_pic/Mybatis/全局配置文件/0.jpg)

![1.jpg](/images/blog_pic/Mybatis/全局配置文件/1.jpg)

# mappers

mappers标签用于管理mapper映射配置文件。例如我们此前已经一再使用的mapper resource标签：

<mappers>
  <mapper resource="mybatis/mapper/UserMapper.xml" />
</mappers>

该标签用于加载类路径下的配置文件。如果要引用完全限定名，则可使用mapper url标签：

<mappers>
  <mapper url="file:///E:\temp\UserMapper.xml" />
</mappers>

还可以使用mapper接口累路径来加载配置文件：

```
<mappers>
  <mapper class="com.user.mapper.UserMapper" />
</mappers>
```

class标签中填入对应mapper接口的全限定名。这种加载方式要求mapper配置文件必须与其对应的mapper接口同名，并在同一目录下(推荐)。

如果要批量加载配置文件，则可使用：

```
<mappers>
  <package name="com.user.mapper" />
</mappers>
```

name标签中传入的即为mapper接口所在的包，这样配置后，该包下的所有mapper接口所对应的配置文件均会被加载。当然，这种加载方式仍然需要mapper配置文件与其对应的mapper接口同名，并在同一目录下(推荐)。

类似于前文对批量别名定义的描述，这种加载方式依然不支持通配符，也就是说，如果想写作：

```
<mappers>
  <package name="com.*.mapper" />
</mappers>
```

是不可以的。如果想添加多个包，只能通过写多个package的方式。

如果与Spring进行了整合，则可以使用整合包中提供的mapper扫描器，彼时mapper的配置方式就会简单得多，也友好得多了。
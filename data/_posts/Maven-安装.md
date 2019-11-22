---
title: Maven-安装
date: 2017-04-23 14:25:08
tags: [Maven,Java]
categories: Maven
---

本文安装方法面向系统：

- Windows7 64位
- Windows10 64位

<!-- more -->

# 安装Maven

**软件下载地址**

- Maven官网下载地址：[下载地址](http://maven.apache.org/download.cgi)。
- Windows64位安装包个人存档：[下载地址](http://pan.baidu.com/s/1pLwgt1h)。

解压即可，不妨将解压后的根目录路径设为：

```
D:\work\java\apache-maven-3.5.0
```

# 配置环境变量

新建变量：

```
变量名: MAVEN_HOME
变量值: D:\work\java\apache-maven-3.5.0
```

在path变量中追加：

```
%MAVEN_HOME%\bin
```

打开cmd，输入：

```
mvn -v
```

返回如下类似信息则安装成功：

![0.jpg](/images/blog_pic/Maven/Maven安装/0.jpg)

# 修改本地仓库存储路径

打开D:\work\java\apache-maven-3.5.0\conf\settings.xml，修改本地仓库路径(路径中不能有空格)：

```
<localRepository>D:\work\java\maven-repository</localRepository>
```

# 关联Maven与Eclipse

打开Eclipse，在菜单项中依次选择：

```
Window --> Preferences --> Maven --> User Settings
```

设置如下:

![1.jpg](/images/blog_pic/Maven/Maven安装/1.jpg)

在菜单项中依次选择：

```
Window --> Preferences --> Maven --> Installations
```

将Eclipse关联的Maven软件改为新下载的版本：

![2.jpg](/images/blog_pic/Maven/Maven安装/2.jpg)

# 配置下载jar包源码

打开D:\work\java\apache-maven-3.5.0\conf\settings.xml，添加：

```
<profiles>  
  <profile>  
    <id>downloadSources</id>  
    <properties>  
      <downloadSources>true</downloadSources>  
      <downloadJavadocs>true</downloadJavadocs>             
    </properties>  
  </profile>  
</profiles>  
  
<activeProfiles>  
  <activeProfile>downloadSources</activeProfile>  
</activeProfiles> 
```

打开Eclipse，在菜单项中依次选择：

```
Window --> Preferences --> Maven
```

配置如下：

![3.jpg](/images/blog_pic/Maven/Maven安装/3.jpg)

# 指定Maven默认Java版本

打开D:\work\java\apache-maven-3.5.0\conf\settings.xml，添加：

```
<profiles> 
  <profile>
    <id>jdk-1.7</id>
    <activation>
      <activeByDefault>true</activeByDefault>
      <jdk>1.7</jdk>
    </activation>
    <properties>
      <maven.compiler.source>1.7</maven.compiler.source>
      <maven.compiler.target>1.7</maven.compiler.target>
      <maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
    </properties>
  </profile>
</profiles>
```

示例中使用的是JDK1.7。依实际需求可改成对应版本。
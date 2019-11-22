---
title: Tomcat-CATALINA_HOME与CATALINA_BASE
date: 2017-12-12 10:22:49
tags: [Tomcat]
categories: Tomcat
---

如果同一台机器上只需要起一个Tomcat实例，那么是不涉及本文要探讨的问题的。不过如果要起多个实例呢？这在日常开发中确实是很常见的需求：例如多个程序员共用一台测试机，他们都需要在上面部署自己的Tomcat(显然将他们的应用都丢到一个Tomcat下是不合理的，因为没人希望启动自身应用时会带着把其他人的应用也启动起来)。

<!-- more -->

一个简单可行的解决方案就是将Tomcat复制为几个独立的副本，然后各自修改Tomcat的启动参数，例如端口，JVM参数等。

这个方法简单粗暴，然而却不那么优雅：因为通常修改的其实仅仅就只有启动参数，而绝大多数的资源，例如jar包，Tomcat的核心配置参数等都是重复的。当然也会有不通常的情况：某个实例要用Tomcat5而另一个要用Tomcat7。不过这个只是小概率事件，绝大多数时候都只需要改配置。

CATALINA_BASE就是为了解决这个问题而诞生的。简单来说，CATALINA_HOME是Tomcat的安装目录，CATALINA_BASE是Tomcat的工作目录。安装目录中存放的是Tomcat中可共享的资源，只有一个。而工作目录则是每个Tomcat实例对应一个，其中只存放自身特殊的配置信息，核心信息还是要去安装目录读取。

举个小例子。我的环境为Windows7，用于测试的Tomcat版本为apache-tomcat-7.0.82。其目录结构如下：

![0.jpg](/images/blog_pic/Tomcat/CATALINA_HOME与CATALINA_BASE/0.jpg)

它的存放路径为

```
D:\test\apache-tomcat-7.0.82
```

我们不妨将其设为CATALINA_HOME，即安装目录。然后我们再在其所在的目录中新建两个目录用于存放Tomcat实例：

![1.jpg](/images/blog_pic/Tomcat/CATALINA_HOME与CATALINA_BASE/1.jpg)

即tomcat1及tomcat2的路径就是对应Tomcat实例的CATALINA_BASE，即工作目录。

现以实例tomcat1为例，将安装目录中每个实例私有的数据：

![2.jpg](/images/blog_pic/Tomcat/CATALINA_HOME与CATALINA_BASE/2.jpg)

复制到目录tomcat1下。

当然为了能够同时启动各个实例，我们还必须为每个实例设置不同的监听端口。其位置在tomcat1目录下的conf/server.xml中。这里我们均保持默认值：

```
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
  </Service>
</Server>
```

为了启动方便，我们还可以在tomcat1目录下添加一个小脚本startup.bat：

```
set "CATALINA_BASE=%cd%"
set "CATALINA_HOME=D:\test\apache-tomcat-7.0.82"
set "EXECUTABLE=%CATALINA_HOME%\bin\catalina.bat"
call "%EXECUTABLE%" start
```

同理，我们可以继续配置实例tomcat2，当然要给它一个不同的端口，例如：

```
<Server port="8006" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8088" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
  </Service>
</Server>
```

依次双击tomcat1及tomcat2目录下的startup.bat。这样这两个实例便同时启动起来了：

![3.jpg](/images/blog_pic/Tomcat/CATALINA_HOME与CATALINA_BASE/3.jpg)

至此，CATALINA_BASE便已介绍完毕了。我本人是很少用这个功能的，即便需要在同一个机器上起多个Tomcat实例，我往往也会采取本文最开始介绍的那种不那么"优雅"的做法。原因主要在于那样简单粗暴，不需要像后者这样配置来配置去。同时损失的不过是一点点磁盘空间而已，通常完全可以接受。
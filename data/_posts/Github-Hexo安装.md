---
title: Github-Hexo安装
date: 2017-04-16 20:49:16
tags: [Hexo,Nodejs]
categories: Github
---

本文安装方法面向系统：Windows7/10 64位。

<!-- more -->

# 安装Node.js

**软件下载地址**

- [Node.js官网下载地址](https://nodejs.org/dist/v4.2.3/node-v4.2.3-x64.msi)
- [Windows64位安装包个人存档](https://pan.baidu.com/s/1dpXtKNvy9CQ2yMmilHFlsA)

安装时，除安装目录外，其余设定保持默认即可。

**验证**

执行：

```
node -v
npm -v
```

安装成功时的结果形如：

![0.jpg](/images/blog_pic/Github/Hexo安装/0.jpg)

# 安装Hexo

**安装**

新建Hexo安装目录并进入到该目录下，执行：

```
npm install hexo-cli -g
```

安装时间可能会较长，期间可能无输出或输出warn信息，为正常情况。安装完成后输入如下语句验证：

```
hexo -v
```

安装成功时输出形如：

![1.jpg](/images/blog_pic/Github/Hexo安装/1.jpg)

**初始化**

进入到Hexo安装目录下，依次执行：

```
hexo init
npm install
```

安装时间可能会较长，期间可能无输出或输出warn信息，为正常情况。
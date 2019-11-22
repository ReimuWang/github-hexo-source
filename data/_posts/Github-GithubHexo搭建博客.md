---
title: Github-Github+Hexo搭建博客
date: 2017-04-11 21:15:33
tags: [Github,Hexo]
categories: Github
---

本文安装方法面向系统：Windows7/10 64位。

<!-- more -->

# Github环境搭建

**git安装**

参见：[Github-Git安装](/2017/04/11/Github-Git安装/)。

**注册Github账号**

登陆[GitHub官网](https://github.com/)进行注册，直到回复确认邮件后才算是注册完成。

**Github添加SSH密钥**

参见：[Github-添加SSH密钥](/2017/04/12/Github-添加SSH密钥/)。

**创建存储博客应用的仓库**

登入Github个人主页，点击下图右上角红框中的位置，创建一个新的仓库用于部署博客应用。

![0.jpg](/images/blog_pic/Github/GithubHexo搭建博客/0.jpg)

创建仓库时只需要填写Repository name，博客应用中该值固定为**Gitgub账号名.github.io**，以我为例，为：

```
ReimuWang.github.io
```

填写完成后点击创建按钮：

![1.jpg](/images/blog_pic/Github/GithubHexo搭建博客/1.jpg)

创建后即可生成用于存储博客的仓库：

![2.jpg](/images/blog_pic/Github/GithubHexo搭建博客/2.jpg)

# Hexo环境搭建

参见：[Github-Hexo安装](/2017/04/16/Github-Hexo安装/)。

# Github与Hexo建立关联

**安装Github-Hexo关联扩展**

进入到Hexo安装目录下，执行：

```
npm install hexo-deployer-git --save
```

**修改Hexo配置文件**

打开Hexo根目录下的_config.yml，修改deploy属性：

```
deploy:
  type: git
  repo: git@github.com:ReimuWang/ReimuWang.github.io.git
  branch: master
```

其中**ReimuWang**为Github账号名。
---
title: Github-添加SSH密钥
date: 2017-04-12 21:35:34
tags: [Github,Git]
categories: Github
---

本文方法面向系统：Windows7/10 64位。

<!-- more -->

SSH密钥分为私钥与公钥两部分:私钥存储在本地机器中，而公钥则上传到Github服务器的自身账号下。

# 本地机器端

**设置Git用户名**

```
git config --global user.name "ReimuWang"
```

**设置Git邮箱**

```
git config --global email.name "1505580759@qq.com"
```

**生成SSH密钥对**

```
ssh-keygen -t rsa -C "1505580759@qq.com"
```

-C后的参数值为上文设置的Git邮箱。命令输入后会提示输入密码，无需密码的情况下，连按3个回车即可。

执行完成后进入~/.ssh，即可发现其下多了两个文件。其中id_rsa为私钥，id_rsa.pub为公钥。

**将生成的私钥注册至本地机器的ssh-agent**

注册前首先确保本地的ssh-agent可用。执行：

```
eval "$(ssh-agent -s)"
```

如果返回值形如：

```
Agent pid 5292
```

则ssh-agent即为可用。此时将刚才生成的私钥注册至本机的ssh-agent中：

```
ssh-add ~/.ssh/id_rsa
```

# Github服务器端

**将公钥注册至Github的个人账户下**

登陆Github，进入个人页。点击右上角的Settings：

![0.jpg](/images/blog_pic/Github/Github添加SSH密钥/0.jpg)

点击左侧的SSH and GPG keys：

![1.jpg](/images/blog_pic/Github/Github添加SSH密钥/1.jpg)

点击右侧的New SSH key：

![2.jpg](/images/blog_pic/Github/Github添加SSH密钥/2.jpg)

将id_rsa.pub文件中的值粘贴至key选框中。然后点击Add SSH key：

![3.jpg](/images/blog_pic/Github/Github添加SSH密钥/3.jpg)

# 验证

执行：

```
ssh -T git@github.com
```

如果结果返回形如：

```
Hi ReimuWang! You've successfully authenticated, but GitHub does not provide shell access.
```

则证明配对成功。
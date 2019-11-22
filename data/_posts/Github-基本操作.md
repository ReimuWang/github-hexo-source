---
title: Github-基本操作
date: 2017-04-22 16:44:45
tags: [Github]
categories: Github
---

# 创建新项目

通过Github页面创建新项目，得到SSH clone地址(使用HTTP时每次提交均需输入账号密码，SSH不需要)。进入选定的仓库目录，clone远端代码：

```
git clone git@github.com:ReimuWang/gitHubTest.git
```

<!-- more -->

# 日常操作流程

触发时机：阶段性修改完成，需提交至远端(首次填充空仓库也可视为一次较大的改动而非创建)。

**拉取最新的代码**

```
git pull
```

**查看当前工作区的更改情况**

```
git status
```

**根据status的指示，将工作区中修改的文件添加至缓存区**

```
git add 文件名
```

**特别的，如下命令将添加所有变更文件**

```
git add *
```

**将当前缓存区中的文件提交到本地HEAD中(该命令执行后还需输入一下本次提交的简介)**

```
git commit
```

**将本地HEAD中的文件提交至远端master分支，可积攒多个commit后统一提交**

```
git push origin master
```
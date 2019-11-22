---
title: Python-Windows下Python安装
date: 2017-04-24 23:57:06
tags: [Python,Pip,Pycharm]
categories: Python
---

本文安装方法面向系统：

- Windows7 64位
- Windows10 64位

<!-- more -->

# 安装Python

**软件下载地址**

- Python官网下载地址：[下载地址](https://www.python.org/ftp/python/2.7.10/python-2.7.10.amd64.msi)。
- Windows64位安装包个人存档：[下载地址](http://pan.baidu.com/s/1mihZpwO)。

安装时，除安装目录选择外，其余设定保持默认即可。

不妨将安装目录设为：

```
D:\work\python\python27
```

# 配置python环境变量

在path变量中追加：

```
D:\work\python\python27
```

打开cmd，输入：

```
python -V
```

返回如下类似信息则安装成功：

![0.jpg](/images/blog_pic/Python/Windows下Python安装/0.jpg)

# 安装Pip

**软件下载地址**

- Pip官网下载地址：[下载地址](https://pypi.python.org/packages/11/b6/abcb525026a4be042b486df43905d6893fb04f05aac21c32c638e939e447/pip-9.0.1.tar.gz#md5=35f01da33009719497f01a4ba69d63c9)。
- Windows64位安装包个人存档：[下载地址](http://pan.baidu.com/s/1boMCZqJ)。

解压后打开cmd，进入解压目录，执行：

```
python setup.py install
```

安装完成后安装包即可删除。

# 配置Pip环境变量

在path变量中追加：

```
D:\work\python\python27\Scripts
```

打开cmd，输入：

```
pip list
```

返回如下类似信息则安装成功：

![1.jpg](/images/blog_pic/Python/Windows下Python安装/1.jpg)

# 安装Pycharm

**软件下载地址**

- Windows64位安装包个人存档：[下载地址](http://pan.baidu.com/s/1pLRo7Mj)。

安装时，除安装目录选择外，其余设定保持默认即可。

# Pycharm注册

首次启动Pycharm时会提示输入注册信息。

解压下载文件中的压缩包PyCharm4.0_KeyGen.zip，得到keygen.exe，执行该文件。将Application切换至PyCharm，得到User Name及License Key。填入对应位置。

# Pycharm设置

**文件编码**

```
File --> File Encoding
```

**缩进**

```
File --> Settings --> Editor --> Code Style --> Python
```

**字号**

```
File --> Settings --> Editor --> Colors & Fonts --> Font
File --> Settings --> Editor --> Colors & Fonts --> Console Font
```
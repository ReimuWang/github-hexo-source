---
title: Python-Centos6.x中Python2.6升级至2.7
date: 2018-03-08 12:28:06
tags: [Python,Pip]
categories: Python
---

本文安装方法面向系统：

- Centos6.x

<!-- more -->

# 确定版本

```
python --version
```

若版本为2.6(Centos6.x中默认自带的Python就是2.6的)则可依本文升级为2.7。

# 下载Python

- [Python-2.7.12官方下载](https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz)
- [Python-2.7.12个人存档](https://pan.baidu.com/s/1bYiJf1VY_rMBEaaHCRz3pg)

# 安装Python2.7

以Python-2.7.12.tgz为例：

```
tar -zxvf Python-2.7.12.tgz
cd Python-2.7.12
./configure  
make all             
make install  
make clean  
make distclean
```

执行：

```
/usr/local/bin/python2.7 -V
```

若输出：

```
Python 2.7.12
```

则说明安装完成，可删除安装包。

此时虽然已完成2.7的安装，但系统默认的Python依然是2.6的。因此需建立软连接，使系统默认的Python指向Python2.7：

```
mv /usr/bin/python /usr/bin/python2.6.6  
ln -s /usr/local/bin/python2.7 /usr/bin/python
```

然后再查看系统默认的Python版本：

```
python -V
```

输出：

```
Python 2.7.12
```

至此系统默认的Python已指向Python2.7。

# 修改yum配置文件

因yum需在Python2.6的环境下运行，因此需将其所用的Python环境单独改回2.6：

```
vim /usr/bin/yum  
将文件头部的
#!/usr/bin/python
改成
#!/usr/bin/python2.6.6
```

# 安装pip

```
wget https://bootstrap.pypa.io/get-pip.py  
python get-pip.py 
```
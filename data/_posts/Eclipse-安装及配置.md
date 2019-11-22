---
title: Eclipse-安装及配置
date: 2017-09-29 10:53:49
tags: [Eclipse]
categories: Eclipse
---

# 下载及安装

<!-- more -->

- [Eclipse官网下载地址](https://www.eclipse.org/downloads/eclipse-packages/)
- [Windows 64位安装包个人存档](https://pan.baidu.com/s/1o39mYkKkb74JfndH84VmnQ)

解压即可完成安装。

# 设置workspace

第一次运行时会提示指定workspace，若此后需要再次修改，则配置：

```
Window—>Preferences—>General—>Startup and Shutdown—>Workspaces—>选中Prompt for workspace on startup
```

删除已有workspace后，重启Eclipse即可重新选workspace。

# 设置编码

```
Window—>Preferences—>General—>workspace
```

将Text file encoding设置为utf-8。

# 设置Java版本

```
Window—>Preferences—>java—>Compiler 将编译器版本设定为所需版本

Window—>Preferences—>java—>Installed JREs 指向对应版本的JDK目录
```

# 设置Maven

见[Maven-安装](/2017/04/23/Maven-安装/)。

# 设置缩进为4个空格

```
Window—>Preferences—>General—>Editors—>Text Editors，选中右侧的insert space for tabs并保存

Window—>Preferences—>java—>code style—>formatter 设置缩进为4个空格
```

# 设置字体

```
Window—>Preferences—>General—>Appearance—>Colors and Fonts 修改Basic下的Text Font
```

# 设置主题

```
Window—>Preferences—>General—>Appearance—>Color Theme 选择主题，个人推荐NightLion Aptana Theme
```

若没有这个选项，则需要先下载插件：

```
Help—>Eclipse Marketplace 搜索并安装Eclipse Color Theme
```
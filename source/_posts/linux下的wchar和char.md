---
title: linux操作系统与Windows操作系统
date: 2019-10-17 11:22:07
tags: linux
toc: true
---

记录一些lLinux操作系统和Windows操作系统下的区别。

<!--more-->

[toc]



## 字符的输出

在VC++（Windows）下，你可以在`swprintf`（或者`wprintf`, `fwprintf`）函数中使用`"%s"`来完成格式化数据的输入。但是在POSIX中，你必须使用`"%ls"`。

|   type   | sprintf | sprintf | swprintf | swprintf |
| :------: | :-----: | :-----: | :------: | :------: |
|          | Windows |  POSIX  | Windows  |  POSIX   |
| ls or lS | wchar_t | wchar_t | wchar_t  | wchar_t  |
|    s     |  char   |  char   | wchar_t  |   char   |
|    S     | wchar_t |  char   |   char   |   char   |


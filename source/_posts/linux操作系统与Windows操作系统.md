---
title: linux操作系统与Windows操作系统
date: 2019-10-17 11:22:07
tags: linux
toc: true
---

记录一些lLinux操作系统和Windows操作系统下的区别。

<!--more-->

[toc]

# 字符

## 字符的输出

在VC++（Windows）下，你可以在`swprintf`（或者`wprintf`, `fwprintf`）函数中使用`"%s"`来完成格式化数据的输入。但是在POSIX中，你必须使用`"%ls"`。

|   type   | sprintf | sprintf | swprintf | swprintf |
| :------: | :-----: | :-----: | :------: | :------: |
|          | Windows |  POSIX  | Windows  |  POSIX   |
| ls or lS | wchar_t | wchar_t | wchar_t  | wchar_t  |
|    s     |  char   |  char   | wchar_t  |   char   |
|    S     | wchar_t |  char   |   char   |   char   |

## 宽字符的宏

在Windows下，若VS中的工程选择的是Unicode编码，那么`_T("abc")`和`__T("abc")`就可以直接将"abc"转换成Unicode编码格式。在将该段代码移植至Linux下时，若只是少量改动还好说。如果代码中大量的使用`_T()`那么可以将其整体通过宏定义的方式替换一下。

```cpp
#define _T(x) L ## x
#define __T(x) _T(x)
```

> 具体请参考 [C++中预处理器运算符](https://lianghm.top/2019/11/08/c-%E4%B8%AD%E9%A2%84%E5%A4%84%E7%90%86%E5%99%A8%E8%BF%90%E7%AE%97%E7%AC%A6/)
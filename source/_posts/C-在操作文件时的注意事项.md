---
title: C++在操作文件时的注意事项
date: 2019-06-28 14:14:02
tags: C++
---
## 按行读取到文件末尾
读取文件中的内容时，有时会使用`std::string::getline()`函数来依此按行读取。在使用该函数的时候，读取到文件末尾时会将文件的iostate中的`eofbit`和`failbit`置位。这时候再进行`ios::seekp`和`ios::seekp`操作时没有用的。可以调用`ios::clear`操作将iostate重置，这样就可以进行`ios::seekp`和`ios::seekp`操作了。
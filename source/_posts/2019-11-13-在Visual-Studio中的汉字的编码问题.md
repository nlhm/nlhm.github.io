---
title: 在Visual Studio中的汉字的编码问题
date: 2019-11-13 13:57:41
tags: C++
---

在VS的编辑器中直接写到char数组中的编码格式是什么呢？

<!--more-->

汉字因为编码长度的问题，不能直接用ASCII编码表示。于是在计算机中存储汉字，大陆编写了针对简体中文的标准GB2312编码，台湾编写了针对繁体中文的BIG5编码。在Windows系统中，简体中文的系统下，ANSI代表的就是GB2312；繁体中文的系统下，ANSI代表的是BIG5。

在ANSI下，一个汉字一般占据两个字节。

> 因为计算机只能处理数字，如果要处理文本，就必须先把文本转换为数字才能处理。最早的计算机在设计时采用8个比特（bit）作为一个字节（byte），所以，一个字节能表示的最大的整数就是255（二进制11111111=十进制255），如果要表示更大的整数，就必须用更多的字节。比如两个字节可以表示的最大整数是`65535`，4个字节可以表示的最大整数是`4294967295`。
>
> 由于计算机是美国人发明的，因此，最早只有127个字符被编码到计算机里，也就是大小写英文字母、数字和一些符号，这个编码表被称为`ASCII`编码，比如大写字母`A`的编码是`65`，小写字母`z`的编码是`122`。
>
> 但是要处理中文显然一个字节是不够的，至少需要两个字节，而且还不能和ASCII编码冲突，所以，中国制定了`GB2312`编码，用来把中文编进去。
>
> 你可以想得到的是，全世界有上百种语言，日本把日文编到`Shift_JIS`里，韩国把韩文编到`Euc-kr`里，各国有各国的标准，就会不可避免地出现冲突，结果就是，在多语言混合的文本中，显示出来会有乱码。
>
>  因此，Unicode应运而生。 Unicode把所有语言都统一到一套编码里，这样就不会再有乱码问题了。
>
> Unicode标准也在不断发展，但最常用的是用两个字节表示一个字符（如果要用到非常偏僻的字符，就需要4个字节）。现代操作系统和大多数编程语言都直接支持Unicode。
>
> 现在，捋一捋ASCII编码和Unicode编码的区别：ASCII编码是1个字节，而Unicode编码通常是2个字节。
>
> 字母`A`用ASCII编码是十进制的`65`，二进制的`01000001`；
>
> 字符`0`用ASCII编码是十进制的`48`，二进制的`00110000`，注意字符`'0'`和整数`0`是不同的；
>
> 汉字`中`已经超出了ASCII编码的范围，用Unicode编码是十进制的`20013`，二进制的`01001110 00101101`。
>
> 你可以猜测，如果把ASCII编码的`A`用Unicode编码，只需要在前面补0就可以，因此，`A`的Unicode编码是`00000000 01000001`。
>
> 新的问题又出现了：如果统一成Unicode编码，乱码问题从此消失了。但是，如果你写的文本基本上全部是英文的话，用Unicode编码比ASCII编码需要多一倍的存储空间，在存储和传输上就十分不划算。
>
> 所以，本着节约的精神，又出现了把Unicode编码转化为“可变长编码”的`UTF-8`编码。UTF-8编码把一个Unicode字符根据不同的数字大小编码成1-6个字节，常用的英文字母被编码成1个字节，汉字通常是3个字节，只有很生僻的字符才会被编码成4-6个字节。如果你要传输的文本包含大量英文字符，用UTF-8编码就能节省空间

```cpp
#include "pch.h"
#include <iostream>
#include <Windows.h>
using namespace std;

int main()
{
	char buff[] = "中国";
	cout << "char * " << buff << " 所占据的字符数目为: "  << strlen(buff) * sizeof(char) << endl;
	int wide_len = MultiByteToWideChar(CP_ACP, 0, buff, -1, nullptr, 0);
	wchar_t* wbuff = new wchar_t[wide_len];
	MultiByteToWideChar(CP_ACP, 0, buff, -1, wbuff, wide_len);
	cout << "wchar_t * ANSI " << buff << " 所占据的字符数目为: "  << wcslen(wbuff) * sizeof(wchar_t) << endl;
	int utf_len = MultiByteToWideChar(CP_UTF8, 0, buff, -1, nullptr, 0);
	wchar_t* utfbuff = new wchar_t[utf_len];
	MultiByteToWideChar(CP_UTF8, 0, buff, -1, utfbuff, utf_len);
	cout << "wchar_t * UTF8 " << buff << " 所占据的字符数目为: "  << wcslen(utfbuff) * sizeof(wchar_t) << endl;
 
}
```

> 输出结果为：
>
> char * 中国 所占据的字符数目为: 4
> wchar_t * ANSI 中国 所占据的字符数目为: 4
> wchar_t * UTF8 中国 所占据的字符数目为: 6
>
> 
>
> 其中strlen(buff) = 4, 即4个char
>
> wide_len = 3,  除去'\n', 即2个wchar_t
>
> utf_len = 4, 除去'\n', 即3个wchar_t

可以看出，直接在编辑器中写入汉字的格式是ANSI的。
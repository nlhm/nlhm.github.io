---
title: (转载)UTF-8 and Unicode FAQ for Unix/Linux
date: 2019-10-17 14:46:07
tags: linux
toc: true
---

> 本文转载自https://blog.csdn.net/edgar_wu/article/details/3903280，原文翻译地址为为 http://www.linuxforum.net/books/UTF-8-Unicode.html 。原文地址为https://www.cl.cam.ac.uk/~mgk25/unicode.html#ucs

[toc]



# UTF-8 and Unicode FAQ

by [Markus Kuhn](http://www.cl.cam.ac.uk/~mgk25/) 

 [中国LINUX论坛](http://www.linuxforum.net/)翻译小组 xLoneStar[译] 2000年2月 

这篇文章说明了在 POSIX 系统 (Linux,Unix) 上使用 Unicode/UTF-8  所需要的信息. 在将来不远的几年里, Unicode 已经很接近于取代 ASCII  与 Latin-1 编码的位置了.  它不仅允许你处理处理事实上存在于地球上的任何语言文字,  而且提供了一个全面的数学与技术符号集,  因此可以简化科学信息交换.

UTF-8 编码提供了一种简便而向后兼容的方法, 使得那种完全围绕  ASCII 设计的操作系统, 比如 Unix, 也可以使用 Unicode. UTF-8 就是 Unix,  Linux 已经类似的系统使用 Unicode 的方式. 现在是你了解它的时候了.

## 什么是 UCS 和 ISO 10646?

国际标准 **ISO 10646** 定义了 **通用字符集 (Universal  Character Set, UCS)**. UCS 是所有其他字符集标准的一个超集.  它保证与其他字符集是双向兼容的. 就是说,  如果你将任何文本字符串翻译到 UCS格式, 然后再翻译回原编码,  你不会丢失任何信息.

UCS 包含了用于表达所有已知语言的字符. 不仅包括拉丁语,希腊语,  斯拉夫语,希伯来语,阿拉伯语,亚美尼亚语和乔治亚语的描述,  还包括中文, 日文和韩文这样的象形文字, 以及 平假名, 片假名,  孟加拉语, 旁遮普语果鲁穆奇字符(Gurmukhi), 泰米尔语, 印.埃纳德语(Kannada),  Malayalam, 泰国语, 老挝语, 汉语拼音(Bopomofo), Hangul, Devangari, Gujarati,  Oriya, Telugu 以及其他数也数不清的语. 对于还没有加入的语言,  由于正在研究怎样在计算机中最好地编码它们,  因而最终它们都将被加入. 这些语言包括 Tibetian, 高棉语, Runic(古代北欧文字),  埃塞俄比亚语, 其他象形文字, 以及各种各样的印-欧语系的语言,  还包括挑选出来的艺术语言比如 Tengwar, Cirth 和 克林贡语(Klingon). UCS  还包括大量的图形的, 印刷用的, 数学用的和科学用的符号,  包括所有由 TeX, Postscript, MS-DOS，MS-Windows, Macintosh, OCR 字体,  以及许多其他字处理和出版系统提供的字符.

ISO 10646 定义了一个 31 位的字符集. 然而, 在这巨大的编码空间中,  迄今为止只分配了前 65534 个码位 (0x0000 到 0xFFFD). 这个 UCS 的 16位子集称为  **基本多语言面 (Basic Multilingual Plane, BMP)**. 将被编码在 16  位 BMP 以外的字符都属于非常特殊的字符(比如象形文字),  且只有专家在历史和科学领域里才会用到它们. 按当前的计划,  将来也许再也不会有字符被分配到从 0x000000 到 0x10FFFF  这个覆盖了超过 100 万个潜在的未来字符的 21 位的编码空间以外去了.  ISO 10646-1 标准第一次发表于 1993 年, 定义了字符集与 BMP  中内容的架构. 定义 BMP 以外的字符编码的第二部分 ISO 10646-2  正在准备中, 但也许要过好几年才能完成.  新的字符仍源源不断地加入到 BMP 中,  但已经存在的字符是稳定的且不会再改变了.

UCS 不仅给每个字符分配一个代码, 而且赋予了一个正式的名字.  表示一个 UCS 或 Unicode 值的十六进制数, 通常在前面加上 "U+",  就象 U+0041 代表字符"拉丁大写字母A". UCS 字符 U+0000 到 U+007F  与 US-ASCII(ISO 646) 是一致的, U+0000 到 U+00FF 与 ISO 8859-1(Latin-1)  也是一致的. 从 U+E000 到 U+F8FF, 已经 BMP  以外的大范围的编码是为私用保留的.

## 什么是组合字符?

UCS里有些编码点分配给了 **组合字符**.它们类似于打字机上的无间隔重音键.  单个的组合字符不是一个完整的字符.  它是一个类似于重音符或其他指示标记, 加在前一个字符后面. 因而,  重音符可以加在任何字符后面. 那些最重要的被加重的字符,  就象普通语言的正字法(orthographies of common languages)里用到的那种, 在  UCS 里都有自己的位置, 以确保同老的字符集的向后兼容性.  既有自己的编码位置,  又可以表示为一个普通字符跟随一个组合字符的被加重字符, 被称为 **预作字符(precomposed  characters)**. UCS 里的预作字符是为了同没有预作字符的旧编码,  比如 ISO 8859, 保持向后兼容性而设的.  组合字符机制允许在任何字符后加上重音符或其他指示标记,  这在科学符号中特别有用, 比如数学方程式和国际音标字母,  可能会需要在一个基本字符后组合上一个或多个指示标记.

组合字符跟随着被修饰的字符. 比如, 德语中的元音变音字符 ("拉丁大写字母A  加上分音符"), 既可以表示为 UCS 码 U+00C4 的预作字符,  也可以表示成一个普通 "拉丁大写字母A" 跟着一个"组合分音符":U+0041  U+0308 这样的组合. 当需要堆叠多个重音符,  或在一个基本字符的上面和下面都要加上组合标记时,  可以使用多个组合字符. 比如在泰国文中,  一个基本字符最多可加上两个组合字符.

## 什么是 UCS 实现级别?

不是所有的系统都需要支持象组合字符这样的 UCS  里所有的先进机制. 因此 ISO 10646 指定了下列三种实现级别:  

- 级别1

  不支持组合字符和 Hangul Jamo 字符 (一种特别的,     更加复杂的韩国文的编码,     使用两个或三个子字符来编码一个韩文音节) 

- 级别2

  类似于级别1, 但在某些文字中, 允许一列固定的组合字符 (例如,     希伯来文, 阿拉伯文, Devangari, 孟加拉语, 果鲁穆奇语, Gujarati, Oriya,     泰米尔语, Telugo, 印.埃纳德语, Malayalam, 泰国语和老挝语).     如果没有这最起码的几个组合字符, UCS 就不能完整地表达这些语言.

- 级别3

  支持所有的 UCS 字符, 例如数学家可以在任意一个字符上加上一个     tilde(颚化符号,西班牙语字母上面的～)或一个箭头(或两者都加).

## 什么是 Unicode?

历史上, 有两个独立的, 创立单一字符集的尝试. 一个是[国际标准化组织(ISO)](http://www.iso.ch/)的 ISO 10646 项目,  另一个是由(一开始大多是美国的)多语言软件制造商组成的协会组织的  [Unicode 项目](http://www.unicode.org/). 幸运的是, 1991年前后,  两个项目的参与者都认识到, 世界不需要两个不同的单一字符集.  它们合并双方的工作成果, 并为创立一个单一编码表而协同工作.  两个项目仍都存在并独立地公布各自的标准, 但 Unicode 协会和 ISO/IEC  JTC1/SC2 都同意保持 Unicode 和 ISO 10646 标准的码表兼容,  并紧密地共同调整任何未来的扩展.

## 那么 Unicode 和 ISO 10646 不同在什么地方?

Unicode 协会公布的 [Unicode  标准](http://www.unicode.org/unicode/standard/standard.html) 严密地包含了 ISO 10646-1 实现级别3的基本多语言面.  在两个标准里所有的字符都在相同的位置并且有相同的名字.

Unicode 标准额外定义了许多与字符有关的语义符号学,  一般而言是对于实现高质量的印刷出版系统的更好的参考. Unicode  详细说明了绘制某些语言(比如阿拉伯语)表达形式的算法,  处理双向文字(比如拉丁与希伯来文混合文字)的算法和  排序与字符串比较 所需的算法, 以及其他许多东西.

另一方面, ISO 10646 标准, 就象广为人知的 ISO 8859 标准一样,  只不过是一个简单的字符集表. 它指定了一些与标准有关的术语,  定义了一些编码的别名, 并包括了规范说明, 指定了怎样使用 UCS  连接其他 ISO 标准的实现, 比如 ISO 6429 和 ISO 2022. 还有一些与 ISO  紧密相关的, 比如 ISO 14651 是关于 UCS 字符串排序的.

考虑到 Unicode 标准有一个易记的名字, 且在任何好的书店里的  Addison-Wesley 里有, 只花费 ISO 版本的一小部分, 且包括更多的辅助信息,  因而它成为使用广泛得多的参考也就不足为奇了. 然而, 一般认为,  用于打印 ISO 10646-1 标准的字体在某些方面的质量要高于用于打印  Unicode 2.0的. 专业字体设计者总是被建议说要两个标准都实现,  但一些提供的样例字形有显著的区别. ISO 10646-1  标准同样使用四种不同的风格变体来显示表意文字如中文,  日文和韩文 (CJK), 而 Unicode 2.0 的表里只有中文的变体.  这导致了普遍的认为 Unicode 对日本用户来说是不可接收的传说,  尽管是错误的.

## 什么是 UTF-8?

首先 UCS 和 Unicode 只是分配整数给字符的编码表.  现在存在好几种将一串字符表示为一串字节的方法.  最显而易见的两种方法是将 Unicode 文本存储为 2 个 或 4  个字节序列的串. 这两种方法的正式名称分别为 UCS-2 和 UCS-4.  除非另外指定, 否则大多数的字节都是这样的(Bigendian convention).  将一个 ASCII 或 Latin-1 的文件转换成 UCS-2 只需简单地在每个 ASCII  字节前插入 0x00. 如果要转换成 UCS-4, 则必须在每个 ASCII  字节前插入三个 0x00.

在 Unix 下使用 UCS-2 (或 UCS-4) 会导致非常严重的问题.  用这些编码的字符串会包含一些特殊的字符, 比如 '/0' 或 '/', 它们在  文件名和其他 C 库函数参数里都有特别的含义. 另外, 大多数使用  ASCII 文件的 UNIX 下的工具, 如果不进行重大修改是无法读取 16  位的字符的. 基于这些原因, 在文件名, 文本文件, 环境变量等地方, **UCS-2**  不适合作为 **Unicode** 的外部编码.

在 ISO 10646-1 [Annex  R](http://www.cl.cam.ac.uk/~mgk25/ucs/ISO-10646-UTF-8.html) 和 [RFC 2279](ftp://ftp.funet.fi/mirrors/nic.nordu.net/rfc/rfc2279.txt)  里定义的 **UTF-8** 编码没有这些问题. 它是在 Unix  风格的操作系统下使用 Unicode 的明显的方法.

UTF-8 有一下特性:  

- UCS 字符 U+0000 到 U+007F (ASCII) 被编码为字节 0x00 到 0x7F (ASCII 兼容).     这意味着只包含 7 位 ASCII 字符的文件在 ASCII 和 UTF-8     两种编码方式下是一样的.
- 所有 >U+007F 的 UCS 字符被编码为一个多个字节的串,     每个字节都有标记位集. 因此, ASCII 字节 (0x00-0x7F)     不可能作为任何其他字符的一部分.
- 表示非 ASCII 字符的多字节串的第一个字节总是在 0xC0 到 0xFD     的范围里, 并指出这个字符包含多少个字节.     多字节串的其余字节都在 0x80 到 0xBF 范围里.     这使得重新同步非常容易, 并使编码无国界,     且很少受丢失字节的影响.
- 可以编入所有可能的 231个 UCS 代码
- UTF-8 编码字符理论上可以最多到 6 个字节长, 然而 16 位 BMP     字符最多只用到 3 字节长.
- Bigendian UCS-4 字节串的排列顺序是预定的.
- 字节 0xFE 和 0xFF 在 UTF-8 编码中从未用到.

下列字节串用来表示一个字符. 用到哪个串取决于该字符在 Unicode  中的序号.

| U-00000000 - U-0000007F: | 0*xxxxxxx*                                                   |
| ------------------------ | ------------------------------------------------------------ |
| U-00000080 - U-000007FF: | 110*xxxxx* 10*xxxxxx*                                        |
| U-00000800 - U-0000FFFF: | 1110*xxxx* 10*xxxxxx* 10*xxxxxx*                             |
| U-00010000 - U-001FFFFF: | 11110*xxx* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx*                  |
| U-00200000 - U-03FFFFFF: | 111110*xx* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx*       |
| U-04000000 - U-7FFFFFFF: | 1111110*x* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx* 10*xxxxxx* |

xxx 的位置由字符编码数的二进制表示的位填入. 越靠右的 x  具有越少的特殊意义.  只用最短的那个足够表达一个字符编码数的多字节串.  注意在多字节串中, 第一个字节的开头"1"的数目就是整个串中字节的数目.

**例如**: Unicode 字符 U+00A9 = 1010 1001 (版权符号) 在 UTF-8  里的编码为:

> 11000010 10101001 = 0xC2 0xA9

而字符 U+2260 = 0010 0010 0110 0000 (不等于) 编码为:

> 11100010 10001001 10100000 = 0xE2 0x89 0xA0

这种编码的官方名字拼写为 UTF-8, 其中 UTF 代表 **U**CS **T**ransformation  **F**ormat. 请勿在任何文档中用其他名字 (比如 utf8 或 UTF_8)  来表示 UTF-8, 当然除非你指的是一个变量名而不是这种编码本身.

## 什么编程语言支持 Unicode?

在大约 1993  年之后开发的大多数现代编程语言都有一个特别的数据类型, 叫做  Unicode/ISO 10646-1 字符. 在 Ada95 中叫 Wide_Character, 在 Java 中叫 char.

ISO C 也详细说明了处理多字节编码和宽字符 (wide characters) 的机制,  1994 年 9 月 [Amendment 1 to ISO C](http://www.lysator.liu.se/c/na1.html)  发表时又加入了更多. 这些机制主要是为各类东亚编码而设计的,  它们比处理 UCS 所需的要健壮得多. UTF-8 是 ISO C  标准调用多字节字符串的编码的一个例子, *wchar_t*  类型可以用来存放 Unicode 字符.

## 在 Linux 下该如何使用 Unicode?

在 UTF-8 之前, 不同地区的 Linux 用户使用各种各样的 ASCII 扩展.  最普遍的欧洲编码是 ISO 8859-1 和 ISO 8859-2, 希腊编码 ISO 8859-7,  俄国编码 KOI-8, 日本编码 EUC 和 Shift-JIS, 等等. 这使得  文件的交换非常困难, 且应用软件必须特别关心这些编码的不同之处.

最终, Unicode 将取代所有这些编码, 主要通过 UTF-8 的形式. UTF-8  将应用在  

- 文本文件 (源代码, HTML 文件, email 消息, 等等)
- 文件名
- 标准输入与标准输出, 管道
- 环境变量
- 剪切与粘贴选择缓冲区
- telnet, modem 和到终端模拟器的串口连接
- 以及其他地方以前用ASCII来表示的字节串

在 UTF-8 模式下, 终端模拟器, 比如 xterm 或 Linux console driver,  将每次按键转换成相应的 UTF-8 串, 然后发送到前台进程的 stdin 里.  类似的, 任何进程在 stdout 上的输出都将发送到终端模拟器,  在那里用一个 UTF-8 解码器进行处理, 之后再用一种 16  位的字体显示出来.

只有在功能完善的多语言字处理器包里才可能有完全的 Unicode  功能支持. 而广泛用在 Linux 里用于取代 ASCII 和其他 8  位字符集的方案则要简单得多. 第一步, Linux  终端模拟器和命令行工具将只是转变到 UTF-8. 这意味着只用到 级别1  的 ISO 10646-1 实现 (没有组合字符),  且只支持那些不需要更多处理的语言象 拉丁, 希腊, 斯拉夫  和许多科学用符号. 在这个级别上, UCS 支持与 ISO 8859 支持类似,  唯一显著的区别是现在我们有几千种字符可以用了,  其中的字符可以用多字节串来表示.

总有一天 Linux 会当然地支持组合字符, 但即便如此,  对于组合字符串, 预作字符(如何可用的话)仍将是首选的. 更正式地,  在 Linux 下用 Unicode 对文本编码的首选的方法应该是定义在 [Unicode Technical Report #15](http://www.unicode.org/unicode/reports/tr15/)  里的 *Normalization Form C.*

在今后的一个阶段,  人们可以考虑增加在日文和中文里用到的双字节字符的支持 (他们相对比较简单),  组合字符支持, 甚至也许对从右至左书写的语言如希伯来文 (他们可不是那么简单的)  的支持. 但对这些高级功能的支持不应该阻碍简单的平板 UTF-8 在  拉丁, 希腊, 斯拉夫和科学用符号方面的快速应用, 以取代大量的欧洲  8 位编码, 并提供一个象样的科学用符号集.

## 我该怎样修改我的软件?

有两种途径可以支持 UTF-8, 我称之为软转换与硬转换. 软转换时,  各处的数据均保存为 UTF-8 形式, 因而需要修改的软件很少.  在硬转换时, 程序将读入的 UTF-8 数据转换成宽字符数组,  以在应用程序内部处理. 在输出时, 再把字符串转换回 UTF-8 形式.

大多数应用程序只用软转换就可以工作得很好了. 这使得将 UTF-8  引入 Unix 成为切实可行的. 例如, 象 cat 和 echo  这样的程序根本不需要修改. 他们仍然可以对输入输出的是 ISO 8859-2  还是 UTF-8 一无所知, 因为它们只是搬运字节流而没有处理它们.  它们只能识别 ASCII 字符和象 '/n' 这样的控制码, 而这在 UTF-8  下也没有任何改变. 因此, 这些应用程序的 UTF-8  编码与解码将完全在终端模拟器里完成.

而那些通过数字节数来获知字符数量的程序则需要一些小修改. 在  UTF-8 模式下, 它们必须不数入 0x80 到 0xBF 范围内的字节,  因为这些只是跟随字节, 它们本身并不是字符. 例如, ls  程序就必须要修改,  因为它通过数文件名中字符数来排放给用户的目录表格布局. 类似地,  所有的假定其输出为定宽字体, 并因此而格式化它们的程序,  必须学会怎样数 UTF-8 文本中的字符数. 编辑器的功能,  如删除单个字符, 必须要作轻微的修改,  以删除可能属于该字符的所有字节. 受影响有编辑器 (vi,emacs,  等等)以及使用 ncurses 库的程序.

Linux 核心使用软转换也可以工作得很好,  只需要非常微小的修改以支持 UTF-8. 大多数处理字符串的核心功能 (例如:  文件名, 环境变量, 等等) 都不受影响. 下列地方也许必须修改:  

- 控制台显示与键盘驱动程序 (另一个 VT100 模拟器)     必须能编码和解码 UTF-8, 必须要起码支持 Unicode 字符集的几个子集.     从 Linux 1.2 起这些功能已经有了.
- 外部文件系统驱动程序, 例如 VFAT 和 WinNT 必须转换文件名字符编码.     UTF-8 已经加入可用的转换选项的列表里了, 因此 mount     命令必须告诉核心驱动程序用户进程希望看到 UTF-8 文件名. 既然 VFAT     和 WinNT 无论如何至少已经用了 Unicode了, 那么 UTF-8     在这里就可以发挥其优势, 以保证转换中无信息损失.
- POSIX 系统的 tty 驱动程序支持一种 "cooked" 模式,     有一些原始的行编辑功能. 为了让字符删除功能工作正常, stty     必须在 tty 驱动程序里设置 UTF-8 模式, 因此它就不会把 0x80 到 0xBF     范围内的跟随字符也数进去了. [Bruno      Haible](http://clisp.cons.org/~haible/) 那里已经有了一些 stty 和核心 tty 驱动 程序的 [Linux 补丁 ](ftp://ftp.ilog.fr/pub/Users/haible/utf8/)了.

## C 对 Unicode 和 UTF-8 的支持

从 GNU glibc 2.1 开始, wchar_t  类型已经正式定为只存放独立于当前 locale 的, 32位的 ISO 10646 值. glibc  2.2 开始将完全支持 ISO C 中的多字节转换函数 (wprintf(),mbstowcs(),等等),  这些函数可以用于在 wchar_t 和包括 UTF-8 在内的任何依赖于 locale  的多字节编码间进行转换.

例如, 你可以写

```cpp
  wprintf(L"Sch鰊e Gre!/n");
```

然后, 你的软件将按照你的用户在环境变量 LC_CTYPE (例如,  en_US.UTF-8 或 de_DE.ISO_8859-1) 中选择的 locale  所指定的编码来打印这段文字. 你的编译器必须运行在与该 C  源文件所用编码相应的 locale 中,  在目标文件中以上的宽字符串将改为 wchar_t 字符串存储. 在输出时,  运行时库将把 wchar_t 字符串转换回与程序执行时的 locale 相应的编码.

注意, 类似这样的操作:

 

```cpp
  char c = L"a"; 
```

只允许从 U+0000 到 U+007F (7 位 ASCII) 范围里的字符. 对于非 ASCII 字符,  不能直接从 wchar_t 到 char 转换.

现在, 象 readline() 这样的函数在 UTF-8 locale 下也能工作了.

## 怎样激活 UTF-8 模式?

如果你的应用程序既支持 8 位字符集 (ISO 8859-*,KOI-8,等等), 也支持  UTF-8, 那么它必须通过某种方法以得知是否应使用 UTF-8 模式.  幸运的是, 在未来的几年里, 人们将只使用 UTF-8,  因此你可以将它作为默认, 但即使如此, 你还是得既支持传统 8  位字符集, 也支持 UTF-8.

当前的应用程序使用许许多多的不同的命令行开关来激活它们各自的  UTF-8 模式, 例如:  

- xterm 命令行选项 "-u8" 和 X resource "XTerm*utf8:1"
- gnat/gcc 命令行选项 "-gnatW8"
- stty 命令行选项 "iutf8"
- mined 命令行选项 "-U"
- xemacs elisp 包裹 以在 UTF-8 和内部使用的 MULE 编码间转换
- vim 'fileencoding' 选项
- less 环境变量 LESSCHARSET=utf-8

记住每一个应用程序的命令行选项或其他配置方法是非常单调乏味的,  因此急需某种标准方法.

如果你在你的应用程序里使用硬转换, 并使用某种特定的 C  库函数来处理外部字符编码和内部使用的 wchar_t  编码的转换工作, 那么 C 库会帮你处理模式切换的问题.  你只需将环境变量 LC_CTYPE 设为正确的 locale, 例如, 如果你使用 UTF-8,  那就是en.UTF-8, 而如果是 Latin-1, 并需要英语的转换, 则设为  en.ISO_8859-1.

然而, 大多数现存软件的维护者选择用软转换来代替, 而不使用 libc  的宽字符函数, 不仅因为它们还未得到广泛应用,  还因为这会使得软件进行大规模修改. 在这种情况下,  你的应用程序必须自己来获知何时使用 UTF-8 模式.  一种方式是做以下工作:

按照环境变量 LC_ALL, LC_CTYPE, LANG 的顺序,  寻找第一个有值的变量. 如果该值包含 UTF-8 子串 (也许是小写或没有"-")  则默认为 UTF-8 模式 (仍然可以用命令行开关来重设),  因为这个值可靠又恰当地指示了 C 库应该使用一种 UTF-8 locale.

提供一个命令行选项 (或者如果是 X 客户程序则用 X resource 的值)  将仍然是有用的, 可以用来重设由 LC_CTYPE  等环境变量指定的默认值.

## 我怎样才能得到 UTF-8 版本的 xterm?

在 [XFree86](http://www.xfree86.org/) 里带的 [xterm](http://www.clark.net/pub/dickey/xterm/xterm.html) 版本最近已经由 [Thomas E. Dickey](http://www.clark.net/pub/dickey/) 加入了支持 UTF-8  的扩展. 使用方法是, 获取 xterm [patch #119](http://www.clark.net/pub/dickey/xterm/xterm.log.html#xterm_119)  (1999-10-16) 或更新版本, 用 "./configure --enable-wide-chars ; make"  来编译, 然后用命令行选项 -u8 来调用 xterm,  使它将输入输出转换为 UTF-8. 在 UTF-8 模式里使用一个 *-ISO10646-1 字体.  当你在 ISO 8859-1 模式里时也可以使用 *-ISO10646-1 字体, 因为 ISO 10646-1  字体与 ISO 8859-1 字体是完全向后兼容的.

新的支持 UTF-8 的 xterm 版本, 以及一些 ISO 10646-1 字体, 将被收录入  XFree86 4.0 版里.

## xterm 支持组合字符吗?

Xterm 当前只支持级别1的 ISO 10646-1, 就是说, 不提供组合字符的支持.  当前, 组合字符将被当作空格字符对待. xterm  将来的修订版很有可能加入某些简单的组合字符支持,  就是仅仅将那个有一个或多个组合字符的基字符加粗 (logical OR-ing).  对于在基线以下的和在小字符上方的重音符来说,  这样处理的结果还是可以接受的.  对于象泰国文字体那样使用特别设计的加粗字符的文字,  这样处理也能工作的很好. 然而, 对于某些字体里,  在较高的字符上方组合上的重音符, 特别是对于 "fixed" 字体族,  产生的结果就不完全令人满意了. 因此, 在可用的地方,  应该继续优先使用预作字符.

## xterm 支持半宽与全宽 CJK 字体吗?

Xterm 当前只支持那种所有字形都等宽的 cell-spaced 的字体.  将来的修订版很有可能为 CJK 语言加入半宽与全宽字符支持, 类似于  kterm 提供的那种. 如果选择的普通字体是 X×Y  象素大小, 且宽字符模式是打开的, 那么 xterm 会试图装入另外的一个 2X×Y  象素大小的字体 (同样的 XLFD, 只是 AVERAGE_WIDTH  属性的值翻倍). 它会用这个字体来显示所有在 [Unicode Technical Report #11](http://www.unicode.org/unicode/reports/tr11/)  里被分配了*East Asian Wide (W)* 或 *East Asian FullWidth (F)*  宽度属性的 Unicode 字符. 下面这个 C 函数用来测试一个 Unicode  字符是否是宽字符并需要用覆盖两个字符单元的字形来显示:

```cpp
  /* This function tests, whether the ISO 10646/Unicode character code   * ucs belongs into the East Asian Wide (W) or East Asian FullWidth   * (F) category as defined in Unicode Technical Report #11. In this   * case, the terminal emulator should represent the character using a   * a glyph from a double-wide font that covers two normal (Latin)   * character cells. */  
int iswide(int ucs)
  {
    if (ucs < 0x1100)
      return 0;

    return
      (ucs >= 0x1100 && ucs <= 0x115f) || /* Hangul Jamo */
      (ucs >= 0x2e80 && ucs <= 0xa4cf && (ucs & ~0x0011) != 0x300a &&
       ucs != 0x303f) ||                     /* CJK ... Yi */
      (ucs >= 0xac00 && ucs <= 0xd7a3) || /* Hangul Syllables */
      (ucs >= 0xf900 && ucs <= 0xfaff) || /* CJK Compatibility Ideographs */
      (ucs >= 0xfe30 && ucs <= 0xfe6f) || /* CJK Compatibility Forms */
      (ucs >= 0xff00 && ucs <= 0xff5f) || /* Fullwidth Forms */
      (ucs >= 0xffe0 && ucs <= 0xffe6);
  }
```

某些 C 库也提供了函数

```cpp
  #include <wchar.h>  
  int wcwidth(wchar_t wc);  
  int wcswidth(const wchar_t *pwcs, size_t n);
```

用来测定该宽字符 wc 或由 pwcs 指向的字符串中的 n  个宽字符码 (或者少于 n 个宽字符码, 如果在 n  个宽字符码之前遇到一个空宽字符的话) 所要求的列位置的数量.  这些函数定义在 Open Group 的 [Single  UNIX Specification](http://www.unix-systems.org/online.html) 里. 一个拉丁/希腊/斯拉夫/等等的字符要求一个列位置,  一个 CJK 象形文字要求两个, 而一个组合字符要求零个.

## 最终 xterm 是否会支持从右到左的书写?

此刻还没有给 xterm 增加从右到左功能的计划.  希伯来与阿拉伯用户因此不得不靠应用程序在将希伯来文与阿拉伯文字符串送到终端前按左方向翻转它们,  换句话说, 双向处理必须在应用程序里完成, 而不是在 xterm 里. 至少,  希伯来与阿拉伯文在预作字形的可用性的形式上,  以及提示表格上的支持, 比 ISO 8859 要有所改进. 现在还远没有决定  xterm 是否支持双向文字以及该怎样工作. [ISO 6429 = ECMA-48](http://www.ecma.ch/stand/ECMA-048.HTM) 和 [Unicode bidi algorithm](http://www.unicode.org/unicode/reports/tr9/)  都提供了可供选择的开始点. 也可以参考 [ECMA Technical
 Report TR/53](http://www.ecma.ch/techrep/E-TR-053.HTM). Xterm 也不处理阿拉伯文, Hangul 或  印度文本的格式化算法, 而且现在还不太清楚在 VT100  模拟器里处理是否可行和值得, 或者应该留给应用软件去处理.  如果你打算在你的应用程序里支持双向文字输出, 看一下 [FriBidi](http://imagic.weizmann.ac.il/~dov/freesw/FriBidi/), Dov Grobgeld 的  Unicode 双向算法的自由实现.

## 我在哪儿能找到 ISO 10646-1 X11 字体?

在过去的几个月里出现了相当多的 X11 的 Unicode 字体,  并且还在快速增多.  

- Markus Kuhn 正和其他许多志愿者一起工作于手动将旧的 

  -misc-fixed-*-iso8859-1

       字体扩展到覆盖所有的欧洲字符表 (拉丁, 希腊, 斯拉夫,     国际音标字母表. 数学与技术符号, 某些字体里甚至有亚美尼亚语,     乔治亚语, 片假名等). 更多信息请参考 

  Unicode fonts and tools for X11

       页. 这些字体将与 XFree86 一起分发. 例如字体 

  ```
    -misc-fixed-medium-r-semicondensed--13-120-75-75-c-60-iso10646-1
  ```
(旧的 xterm 的 fixed 缺省字体的一个扩展, 包括超过 3000     个字符) 已经是 XFree86 3.9 snapshot 的一部分了.

- Markus 也做好了 [X11R6.4      distribution 里所有的 Adobe 和 B&H BDF 字体的 ISO 10646 版本](http://www.cl.cam.ac.uk/~mgk25/download/ucs-fonts-75dpi100dpi.tar.gz).     这些字体已经包含了全部 Postscript 字体表 (大约 30 个额外的字符,     大部分也被 CP1252 MS-Windows 使用, 如 smart quotes, dashes 等), 在 ISO 8859-1     编码下是没有的. 它们在 ISO 10646-1 版本里是完全可用的.
- XFree86 4.0 将携带一个集成的 TrueType 字体引擎, 这使得你的 X     应用程序可以将任何 Apple/Microsoft 字体用于 ISO 10646-1 编码.
- 将来的 XFree86 版本很有可能从分发版中去除大多数旧的 BDF 字体,     取而代之的是 ISO 10646-1 编码的版本. X     服务器则会增加一个自动编码转换器, 只有当旧的 8     位软件请求一个类似于 ISO 8859-* 编码的字体时, 才虚拟地从 ISO 10646-1     字体文件中创建一个这样的字体. 现代软件应该优先地直接使用 ISO     10646-1 字体编码.
- [ClearlyU (cu12)](ftp://crl.nmsu.edu/CLR/multiling/unicode/fonts/)     是一个非常有用的 X11 的 12 点阵, 100 dpi 的 proportional ISO 10646-1 BDF     字体, 包含超过 3700 个字符, 由 [Mark      Leisher](mailto:mleisher@crl.nmsu.edu) 提供 ([样例图象](http://crl.nmsu.edu/~mleisher/cu-examples.html)).
- Roman Czyborra 的 [GNU Unicode font](http://czyborra.com/unifont/)     项目工作于收集一个完整的与免费的 8×16/16×16 pixel Unicode 字体.     目前已经覆盖了 34000 个字符. 
- [etl-unicode](ftp://ftp.x.org/contrib/fonts/etl-unicode.tar.gz) 是一个 ISO     10646-1 BDF 字体, 由 [Primoz      Peterlin](mailto:primoz.peterlin@biofiz.mf.uni-lj.si) 提供.

Unicode X11 字体名字以 -ISO10646-1 结尾. 这个 [X  逻辑字体描述器 (X Logical Font Descriptor, XLFD)](ftp://sunsite.doc.ic.ac.uk/packages/X11/pub/R6.4/xc/doc/hardcopy/XLFD/xlfd.PS.gz) 的 CHARSET_REGISTRY 和  CHARSET_ENCODING 域里的值已经为所有 Unicode 和 ISO 10646-1 的 16  位字体而正式地[注册](ftp://sunsite.doc.ic.ac.uk/packages/X11/pub/R6.4/xc/registry)了. 每个 *-ISO10646-1  字体都包含了整个 Unicode 字符集里的某几个子集,  而用户必须弄清楚他们选择的字体覆盖哪几个他们需要的字符子集.

*-ISO10646-1 字体通常也指定一个 DEFAULT_CHAR 值,  指向一个非 Unicode 字形, 用来表示所有在该字体里不可用的字符 (通常是一个虚线框,  一个 H 的大小, 位于 0x1F 或 0xFFFE).  这使得用户至少能知道这儿有一个不支持的字符. xterm  用的小的定宽字体比如 6x13 等, 将永远无法覆盖所有的 Unicode,  因为许多文字比如日本汉字只能用比欧洲用户广泛使用的大的象素尺寸才能表示.  欧洲使用的典型的 Unicode 字体将只包含大约 1000 到 3000 个字符的子集.

## 我怎样才能找出一个 X 字体里有哪些字形?

X 协议无法让一个应用程序方便地找出一个 cell-spaced  字体提供哪些字形, 它没有为字体提供这样的量度. 因此 [Mark Leisher](http://crl.nmsu.edu/~mleisher/) 和 [Erik van de Poel](mailto:erik@netscape.com) (Netscape) 指定了一个新的 _XFREE86_GLYPH_RANGES  BDF 属性, 告诉应用程序该 BDF 字体实现了哪个 Unicode 子集. Mark  Leisher 提供了一些[样例代码](http://crl.nmsu.edu/~mleisher/bdfother.html)以产生并扫描这个属性,  而 Xmbdfed 3.9 以及更高版本将自动将其加入到由它产生的每个 BDF  文件里.

## 与 UTF-8 终端模拟器相关的问题是什么?

VT100 终端模拟器接受 ISO 2022 (=[ECMA-35](ftp://ftp.ecma.ch/ECMA-ST/E035-PDF.PDF))  ESC 序列, 用于在不同的字符集间切换.

UTF-8 在 ISO 2022 的意义里是一个 "其他编码系统" (参考 ECMA 35  的 15.4 节). UTF-8 是在 ISO 2022 SS2/SS3/G0/G1/G2/G3 世界之外的,  因此如果你从 ISO 2022 切换到 UTF-8, 所有的 SS2/SS3/G0/G1/G2/G3  状态都变得没有意义了, 直到你离开 UTF-8 并切换回 ISO 2022. UTF-8  是一个没有国家的编码,  也就是一个自我终结的短字节序列完全决定了它代表什么字符,  独立于任何国家的切换. G0 与 G1 在 ISO 10646 里与在 ISO 8859-1 里相同,  而 G2/G3 在 ISO 10646 里不存在, 因为任何字符都有固定的位置,  因而不会发声切换. 在 UTF-8 模式下,  你的终端不会因为你偶然地装入一个二进制文件而切换入一种奇怪图形字符模式.  这使得一个终端在 UTF-8 模式下比在 ISO 2022 模式下要健壮得多,  而且因此可以有办法将终端锁在 UTF-8 模式里, 而不会偶然地回到 ISO  2022 世界里.

ISO 2022 标准指定了一系列的 ESC % 序列, 以离开 ISO 2022 世界 (指定其他的编码系统,  DOCS), 用于 [UTF-8](ftp://ftp.informatik.uni-erlangen.de/pub/doc/ISO/charsets/ISO-10646-UTF-8.html)  的许多这样的序列已经注册进了 [ISO  2375 International Register of Coded Character Sets](http://www.itscj.ipsj.or.jp/ISO-IR/):  

- ESC %G 从 ISO 2022 里激活一个未指定实现级别的 UTF-8     模式且允许再返回 ISO 2022.
- ESC %@ 从 UTF-8 回到 ISO 2022, 条件是通过 ESC %G 进入的 UTF-8
- ESC %/G 切换进 UTF-8 级别 1 且不返回.
- ESC %/H 切换进 UTF-8 级别 2 且不返回.
- ESC %/I 切换进 UTF-8 级别 3 且不返回.

当一个终端模拟器在 UTF-8 模式时, 任何 ISO 2022  逃脱码序列例如用于切换 G2/G3 等的都被忽略. 一个在 UTF-8  模式下的终端模拟器唯一会执行的 ISO 2022 序列是 ESC %@  以从 UTF-8 返回 ISO 2022 方案.

UTF-8 仍然允许你使用象 CSI 这样的 C1 控制字符, 尽管 UTF-8 也使用  0x80-0x9F 范围里的字节. 重要的是必须理解在 UTF-8  模式下的终端模拟器必须在执行任何控制字符前对收到的字节流运用  UTF-8 解码器. C1 字符与其他任何大于 U+007F 的字符一样需先经过 UTF-8  解码.

## 已经有哪些支持 UTF-8 的应用程序了?

- [Yudit](http://czyborra.com/yudit/) 是 [Gaspar Sinai](http://www2.gol.com/users/gsinai/) 的自由 X11 Unicode 编辑器
- [Mined 98](http://www.inf.fu-berlin.de/~wolff/mined.html) 由 [Thomas Wolff](http://www.inf.fu-berlin.de/~wolff/) 提供, 是一个可以处理     UTF-8 的文本编辑器.
- [less](http://www.flash.net/~marknu/less/) 版本 346 或更高, 支持 UTF-8
- [C-Kermit 7.0](http://www.columbia.edu/kermit/ckermit.html) 在传输, 终端,     及文件字符集方面支持 UTF-8.
- [Sam](http://hawkwind.utcs.utoronto.ca:8001/mlists/sam.html) 是 Plan9 的     UTF-8 编辑器, 类似于 vi, 也可用于 Linux 和 Win32. ([Plan9](ftp://ftp.informatik.uni-erlangen.de/pub/doc/ISO/charsets/UTF-8-Plan9-paper.ps.gz)     是第一个完全转向 UTF-8, 将其作为字符编码的操作系统.)
- [9term](http://www.gh.cs.usyd.edu.au/~matty/9term/) 由 [Matty Farrow](http://www.gh.cs.usyd.edu.au/~matty/) 提供, 是一个 Plan9     操作系统的 Unicode/UTF-8 终端模拟器的 Unix 移植.
- [Wily](http://www.cs.su.oz.au/~gary/hobby/wily/auug.html) 是一个 Plan9     Acme 编辑器的 Unix 实现.
- [ucm-0.1](ftp://ftp.dcs.ed.ac.uk/pub/jec/programs/) 是 [Juliusz Chroboczek](http://www.dcs.ed.ac.uk/home/jec/) 的 Unicode     字符映射表, 一个小工具, 使你可以选中任何一个 Unicode     字符并粘贴进你的应用程序.

## 有哪些用于改善 UTF-8 支持的补丁?

- [Robert Brady](http://www.ecs.soton.ac.uk/~rwb197/) 提供了一个 [patch for less 340](http://www.ecs.soton.ac.uk/~rwb197/utf8/less-340-utf-2.diff)     (现在已经合并进了 [less      344](http://www.flash.net/~marknu/less/less-344.tar.gz)) 
- [Bruno Haible](http://clisp.cons.org/~haible/) 提供了用于 stty, Linux     核心 tty 等的 [多个补丁](ftp://ftp.ilog.fr/pub/Users/haible/utf8/).
- Otfried Cheong 编写了 [Unicode      encoding for GNU Emacs](http://www.cs.ust.hk/faculty/otfried/Mule/) 工具箱, 使 Mule 能够处理 UTF-8 文件. 

## Postscript 字形的名字与 UCS 代码是怎么关联的?

参考 Adobe 的 [Unicode and Glyph  Names](http://partners.adobe.com/asn/developer/typeforum/unicodegn.html) 指南. 

## X11 的剪切与粘贴工作在 UTF-8 时是如何完成的?

参考 [Juliusz Chroboczek](http://www.dcs.ed.ac.uk/home/jec/) 的 [客户机间  Unicode 文本的交换](http://www.dcs.ed.ac.uk/home/jec/programs/xfsft/UTF8-selections.text) 草案, 对 ICCCM 的一个扩充的建议,  用一个新的可用于属性类型(property type)和选中(selection)目标的原子  UTF8_STRING 来处理 UTF-8 的选中.

## 现在有没有用于处理 Unicode 的免费的库?

- [IBM Classes for Unicode](http://www.alphaworks.ibm.com/tech/icu/) 
- [Mark Leisher](http://crl.nmsu.edu/~mleisher/) 的 UCData Unicode     字符属性库和 wchar_t 支持测试码.

## 各种 X widget 对 Unicode 支持的现状如何?

- [GScript - Unicode      与复杂文本处理](http://www.labs.redhat.com/~otaylor/gscript/) 是一个为 [GTK+](http://www.gtk.org/)     增加全功能的 Unicode 支持的项目.
- [Qt 2.0](http://www.troll.no/announce/qt-200.html) 现在支持使用     *-ISO10646-1 字体了.
- [FriBidi](http://imagic.weizmann.ac.il/~dov/freesw/FriBidi/) 是 Dov Grobgeld     的 Unicode 双向算法的免费实现. 

## 有什么关于这个话题的好的邮件列表?

你确实应该订阅的是 unicode@unicode.org 邮件列表,  这是发现标准的作者和其他许多领袖的话语的最好办法. 订阅方法是,  用 "subscribe" 作为标题, "subscribe YOUR@EMAIL.ADDRESS unicode"  作为正文, 发一条消息到 [unicode-request@unicode.org](mailto:unicode-request@unicode.org).

也有一个专注与改进通常用于 GNU/Linux 系统上应用程序的 UTF-8  支持的邮件列表 linux-utf8@nl.linux.org. 订阅方法是, 以  "subscribe linux-utf8" 为内容, 发送消息到 [majordomo@nl.linux.org](mailto:majordomo@nl.linux.org). 你也可以浏览 [linux-utf8 archive](http://www.linux.eu.org/lists/linux-utf8/)

其他相关的还有 [XFree86](http://www.xfree86.org/) 组的 "字体"  与 "i18n" 列表, 但你必须成为一名正式的开发者才能订阅.



## 更多参考

- Bruno Haible 's [Unicode      HOWTO](ftp://ftp.ilog.fr/pub/Users/haible/utf8/Unicode-HOWTO.html). 
- [The Unicode Standard, Version      2.0](http://www.unicode.org/unicode/uni2book/u2.html) 
- [Unicode Technical      Reports](http://www.unicode.org/unicode/reports/techreports.html) 
- Mark Davis' [Unicode FAQ](http://www.unicode.org/unicode/faq/) 
- [ISO/IEC 10646-1:1993](http://www.iso.ch/cate/d18741.html) 
- [Frank Tang's I?t?rnati?nàliz?ti?n      Secrets](http://people.netscape.com/ftang/i18n.html) 
- [Unicode Support in the      Solaris 7 Operating Environment](http://www.sun.com/software/white-papers/wp-unicode/) 
- The USENIX paper by Rob Pike and Ken Thompson on the [introduction      of UTF-8 under Plan9](ftp://ftp.informatik.uni-erlangen.de/pub/doc/ISO/charsets/UTF-8-Plan9-paper.ps.gz) reports about the first operating system that migrated already in     1992 completely to UTF-8 (which was at the time still called UTF-2). 
- [Li18nux](http://www.li18nux.org/) is a project initiated by several Linux     distributors to enhance Unicode support for Linux. 
- The [Online Single Unix Specification](http://www.unix-systems.org/online.html)     contains definitions of all the ISO C Amendment 1 function, plus extensions such as     wcwidth(). 
- The Open Group's summary of [ISO C Amendment 1](http://www.unix-systems.org/version2/whatsnew/login_mse.html).   
- [GNU libc](http://sourceware.cygnus.com/glibc/) 
- [The Linux Console Tools](http://www.multimania.com/ydirson/en/lct/) 
- The Unicode Consortium [character      database](ftp://ftp.unicode.org/Public/UNIDATA/) and [character set conversion      tables](ftp://ftp.unicode.org/Public/MAPPINGS/) are an essential resource for anyone developping Unicode related tools. 
- Other conversion tables are available from [Microsoft](http://www.microsoft.com/globaldev/reference/WinCP.asp) and [Keld Simonsen's WG15 archive](ftp://dkuug.dk/i18n/WG15-collection/charmaps/). 
- Michael Everson's [ISO10646-1      archive](http://www.indigo.ie/egt/standards/iso10646/pdf/) contains online versions of many of the more recent ISO 10646-1 amendments,     plus many other goodies. See also his [Roadmaps to the      Universal Character Set](http://www.indigo.ie/egt/standards/iso10646/ucs-roadmap.html). 
- An introduction into [The Universal      Character Set (UCS)](http://www.stri.is/TC304/guidecharactersets/guideannexb.html). 
- Otfried Cheong's essey on [Han      Unification in Unicode](http://www.cs.ust.hk/~otfried/Mule/unihan.html) 
- The [AMS STIX](http://www.ams.org/STIX/) project is working on revising and     extending the mathematical characters for Unicode 4.0 and ISO 10646-2. 
- Jukka Korpela's [Soft hyphen (SHY) - a      hard problem?](http://www.hut.fi/~jkorpela/shy.html) is an excellent discussion of the controversy surrounding U+00AD. 
- James Briggs' [Perl, Unicode and I18N FAQ](http://rf.net/~james/perli18n.html).   

我不断地将新的材料加入这份文档, 因此请定期来查看.  欢迎所有关于改进的[建议](mailto:Markus.Kuhn@cl.cam.ac.uk),  以及自由软件社区里关于改善 UTF-8 支持的广告. UTF-8 用在 Linux  里是新近的事, 因此我们在将来的几个月里可以见到大量的进展.

特别感谢 Ulrich Drepper 和 Bruno Haible 的有价值的注解

[Markus Kuhn](http://www.cl.cam.ac.uk/~mgk25/) <<Markus.Kuhn@cl.cam.ac.uk>
创建于 1999-06-04 -- 最近更新于 2000-01-15 --  http://www.cl.cam.ac.uk/~mgk25/unicode.html 





# Linux Unicode 编程

> 联动阅读，原文地址为https://www.ibm.com/developerworks/cn/linux/i18n/unicode/linuni/index.html

如何（在程序中）加入并使用 Unicode 以实现外语支持 

Unicode 并不只是一个编程工具，它还是一个政治的、经济的工具。没有结合世界的语言支持的应用程序通常只能被那些能读写 ASCII 所支持语言的个人使用。这使得建立在 ASCII 基础之上的计算机技术脱离了世界上大部分人。Unicode 允许程序使用世界上任何一种字符集，因此它支持所有语言。

Unicode 让程序员为普通人提供用他们本国语言就能使用的软件。这样就不用再学一门外语了，而且更容易实现计算机技术社会和财政上的利益。很容易设想，如果用户必须为使用因特网浏览器而学习乌尔都语的话， 您就难以看到计算机在美国的使用。Web 就更不会出现了。

Linux 承担了对 Unicode 很大程度上的支持。Unicode 支持被嵌入到内核和代码开发库中。在很大程度上，使用程序中几句简单的命令就能将它们自动的结合到代码中。

所有现代字符集的基础都是在  1968 年以 ANSIX3.4 版本出版的美国信息交换标准码（American Standard Code for Information Interchange，ASCII）。一个值得注意的例外是在 ASCII 之前定义的 IBM 的扩充的二进制编码的十进制交换码（Extended Binary Coded Decimal Information Code，EBCDIC）。ASCII 是一个编码字符集（coded character set，CCS），换句话说，它是整数到字符表示的映射。ASCII 编码字符集允许用一个八位（基于二进制的，用值 0 或 1 表示的）字段或字节（2^8 =256）表示 256 个字符。 这是一个高度受限的编码字符集，它不能表示许多不同语言的所有字符（如中文和日文），不能表示科学符号，更不能表示古代文字（神秘符号和象形文字）和音乐符号。通过更改一个字节的长度而使更大的字符集得以被编码，这似乎有效但完全不切实际。所有的计算机都基于八位字节。解决方法是一种字符编码方案（Character encoding scheme，CES）― 用定长或变长的多字节序列能够表示比 256 大的数.这些数值接着通过编码字符集被映射到它们表示的字符。

## Unicode 的定义

 Unicode 通常用作涉及双字节字符编码方案的通用术语。Unicode CCS 3.1 的官方称谓是 ISO10646-1 通用多八字节编码字符集（Universal Multiple Octet Coded Character Set，UCS）。Unicode 3.1 版本添加了 44,946 个新的编码字符。算上 Unicode 3.0 版本已经存在的 49,194 个字符，共计 94,140 个。

Unicode 编码字符集利用了一个由 128 个三维的组构成的四维编码空间。其中每个组包含 256 个二维平面。每个平面由 256 个一维的行组成，并且每个行有 256 个单元。每个单元在这个编码空间内对一个字符编码，或者被声明为未经使用。这种编码概念被称为 UCS-4；四个八位元用来表示指定组、平面、行和单元的每个字符。

第一个平面（第 00 组的第 00 平面）是基本多语言平面（Basic Multilingual Plane，BMP）。BMP 按字母、音节、表意符号和各种符号及数字定义了常规使用的字符。后续的平面用于附加字符或其它还没有发明的编码实体。我们需要这完整的范围去处理世界上的所有语言；特别是拥有将近 64,000 个字符的一些东亚语言。

BMP 被用作双字节的编码字符集，这种编码字符集确定为 ISO 10646 UCS-2 格式。ISO 10646 UCS-2 就是指 Unicode（并且两者相同）。BMP，像所有 UCS 平面那样，包含了 256 行，其中每行包含 256 个单元，字符仅仅按照 BMP 中的行和单元的八位元在单元中被编码。 这就允许 16 位编码字符能够被用来书写大多数商业上最重要的语言。UCS-2 不需要代码页切换、代码扩展或代码状态。UCS-2 是一种将 Unicode 结合到软件中的简单方法，但它只限于支持 Unicode BMP。

若要用 8 位字节表示一个多于 2^8 =256 个字符的字符编码系统（character coding system，CCS），就需要一种字符编码方案(character-encoding scheme，CES）。

## Unicode 转换

 在 UNIX 中，使用得最多的字符编码方案是 UTF-8。 它考虑到了对整个 Unicode 全部页和平面的全面支持，而且它仍能正确的识别 ASCII。除了 UTF-8 的其他选择还有：UCS-4、UTF-16、UTF-7.5、UTF-7、SCSU、HTML 和 JAVA。

Unicode 转换格式（Unicode Transformation Formats，UTFs）是一种通过映射多字节编码中的值来支持 Unicode 的字符编码方案。本文将分析最流行的格式 ― UTF-8 字符编码系统。

### UTF-8

 UTF-8 转换格式正逐步成为一种占主导地位的交换国际文本信息的方法，因为它可以支持世界上所有的语言，而且它还与 ASCII 兼容。UTF-8 使用变长编码。从 0 到 0x7f（127）的字符把自身编码成单字节，而将值更大的字符编码成 2 到 6 个字节。

##### 表 1. UTF-8 编码

| 0x00000000 - 0x0000007F: | 0              *xxxxxxx*                                     |
| ------------------------ | ------------------------------------------------------------ |
| 0x00000080 - 0x000007FF: | 110              *xxxxx*10              *xxxxxx*             |
| 0x00000800 - 0x0000FFFF: | 1110              *xxxx*10              *xxxxxx*10              *xxxxxx* |
| 0x00010000 - 0x001FFFFF: | 11110              *xxx*10              *xxxxxx*10              *xxxxxx* 10              *xxxxxx* |
| 0x00200000 - 0x03FFFFFF: | 111110              *xx*10              *xxxxxx*10              *xxxxxx*10              *xxxxxx* 10              *xxxxxx* |
| 0x04000000 - 0x7FFFFFFF: | 1111110              *x*10              *xxxxxx*10              *xxxxxx*10              *xxxxxx* 10              *xxxxxx*10              *xxxxxx* |

字节 10        *xxxxxx*是一个扩展字节，它的        *xxxxxx* 位位置被以二进制表示的字符代码号的位所填充。这是能够代表被使用代码的最短的可能的多字节序列。      

### UTF-8 编码示例

 Unicode 字符版权标记字符 0xA9 = 1010 1001 用 UTF-8 编码如下所示：

> ```
> 11000010 10101001 = 0xC2 0xA9
> ```

“不等于”符号字符 0x2260 = 0010 0010 0110 0000 编码如下所示：

> ```
> 11100010 10001001 10100000 = 0xE2 0x89 0xA0
> ```

通过获取         `continuation byte` 的值可以看到原始数据：      

> ```
> [1110]0010 [10]001001 [10]100000           0010 001001 100000           0010 0010 0110 0000 = 0x2260        
> ```

第一个字节定义后面紧跟的八位元数，如果是 7F 或更小，这就是等价的 ASCII 值。每个八位字节以 10        *xxxxxx* 开头，确保字节不与 ASCII 的值混淆。      

## UTF 支持

 在 Linux 平台上使用 UTF-8 之前，请确信分发包里有 glibc 2.2 和 XFree86 4.0 或更新的版本。早先的版本缺少 UTF-8 语言环境支持和 ISO10646-1 X11 字体。

在 UTF-8 发布之前，Linux 用户使用各种不同特定语言的扩展 ASCII，像欧洲用户用 ISO 8859-1 或 ISO 8859-2，希腊用户使用 ISO 8859-7，俄罗斯用户使用 KOI-8 / ISO 8859-5/CP1251（西里尔字母）。这使得数据交换出现了很多问题，并且需要为这些编码之间的差异编写应用软件。这种语言支持是不完善的，而且数据交换没有经过测试。Linux 主要的发行商和应用程序开发者正致力于让主要以 UTF-8 格式表示的 Unicode 成为 Linux 中的标准。

为了识别 Unicode 文件，Microsoft 建议所有的 Unicode 文件应该以 ZERO WIDTH NOBREAK SPACE（U+FEFF）字符开头。这作为一个“特征符”或“字节顺序标记（byte-order mark，BOM）”来识别文件中使用的编码和字节顺序。但是，Linux/UNIX 并没有使用 BOM，因为它会破坏现有的 ASCII 文件的语法约定。在 POSIX 系统中，选中的语言环境识别了在一个过程中的所有输入输出文件期望的编码形式。

有两种方法可以将 UTF-8 支持添加到 Linux 应用程序中。第一种方法，数据都以 UTF-8 形式存放在各处，这样软件改动很少（被动的）。另一种方法，被读取的 UTF-8 数据用标准的 C 语言库函数转变成为宽字符数组（转换的）。在输出时，用函数         `wcsrtombs()` 使字符串被转变回 UTF-8：      

##### 清单 1. wcsrtombs()

```cpp
#include <wchar.h> 
size_t wcsrtombs (char *dest, const wchar_t **src, size_t len, mbstate_t *ps);
```

 方法的选择取决于应用程序的性质。大多数应用程序可以使用被动的方法操作。这就是在 UNIX 平台上使用 UTF-8 会如此流行的原因。像         `cat` 和         `echo`  那样的程序就不需要修改。字节流仍只是字节流，并没有对它进行任何处理。ASCII 字符和控制代码在 UTF-8 语言环境中不改变。      

通过字节计数对字符进行计数的程序需要一些小小的改动。在 UTF-8 中应用程序不对任何扩展的字节进行计数。如果选择了 UTF-8 语言环境，C 语言库的         `strlen(s)` 函数需要用         `mbstowcs()`  函数来代替：      

##### 清单 2. mbstowcs() 函数

```cpp
#include <stdlib.h>
size_t mbstowcs(wchar_t *pwcs, const char *s, size_t n);
```

`strlen`  的一种常见用法是估算显示宽度。中文和其它表意符号将占用两列位置。         ` wcwidth()` 函数用来测试每个字符的显示宽度：      

##### 清单 3. wcwidth() 函数

```cpp
#include <wchar.h> 
int wcwidth(wchar_t wc);
```

## Unicode 的 C 语言支持

在正式情况下，从 GNU glibc 2.2 开始，wchar_t 类型只为 32 位的 ISO 10646 格式数值所特定使用，与当前使用的语言环境无关。通过 ISO C99 所要求的 __STDC_ISO_10646__ 宏的定义作为信号通知应用程序。 __STDC_ISO_10646__ 的定义用来指出 wchar_t 是 Unicode。精确的值是一个十进制的 yyyymmL 格式的常数。例如，使用：

##### 清单 4. 指出 wchar_t 是 Unicode

```cpp
#define __STDC_ISO_10646__ 200104L
```

是为指出 wchar_t 类型的值是由 ISO/IEC 10646 和到指定的年月为止的所有修正与技术勘误定义的字符编码表示。

对 wchar_t 的利用如这个示例所示，使用宏确定在 ISO C99 可移植代码中写双引号的方法。

##### 清单 5. 确定写双引号的方法

```cpp
#if __STDC_ISO_10646__  
printf("%lc", 0x201c); 
#else
putchar('"'); 
#fi
```

### 语言环境

 激活 UTF-8 的恰当的办法是 POSIX 语言环境机制。语言环境是一种包含有关软件行为特定文化约定的配置设定。它包含了字符编码、日期／时间符号、分类规则以及度量系统。语言环境的名称通常由 ISO 639-1 语言、ISO 3166-1 国家或地区代码以及可选的编码名称和其它限定符组成。您可以用命令         `locale -a` 获取所有安装在系统上的语言环境列表（通常在 /usr/lib/locale/）。      

如果没有预安装 UTF-8 语言环境，你可以用         `localedef` 命令生成它。若要为某个特定用户生成并激活一个德语的 UTF-8 语言环境，请使用如下语句：      

##### 清单 6. 为特定用户生成语言环境

```cpp
localedef -v -c -i de_DE -f UTF-8 $HOME/local/locale/de_DE.UTF-8
export LOCPATH=$HOME/local/locale
export LANG=de_DE.UTF-8
```

有时候为所有用户添加 UTF-8 语言环境会很有用。root 用户使用如下指令就可以完成：

##### 清单 7. 为每个用户生成语言环境

```cpp
localedef -v -c -i de_DE -f UTF-8 /usr/share/locale/de_DE.UTF-8
```

若要为每个用户将这个语言环境设为缺省值，可以将以下行添加到 /etc/profile 文件中：

##### 清单 8. 为所有用户设置缺省的语言环境

```cpp
export LANG=de_DE.UTF-8
```

处理多字节字符代码序列的函数行为依赖于当前语言环境的 LC_CTYPE 类别；它确定了依赖语言环境的多字节编码。值 LANG=de_DE（德语）会导致输出按 ISO 8859-1 被格式化。值 LANG=de_DE.UTF-8 会把输出格式化成 UTF-8。语言环境设置会导致         `printf` 中的         `%ls` 格式说明符调用         `wcsrtombs()`  函数以便于将宽字符的参数字符串转换成依赖语言环境的多字节编码。语言环境中的国家或地区标识符如：LC_CTYPE= en_GB （英国英语）和 LC_CTYPE= en_AU（澳大利亚英语），它们之间的差异只在 LC_MONETARY 类别中，原因在于货币的名称和打印货币数量的规则不同。      

请给您首选的语言环境设置环境变量 LANG。当一个 C 程序执行         `setlocale()` 函数时：      

##### 清单 9. setlocale() 函数

```cpp
#include <stdio.h>
#include <locale.h>
//char *setlocale(int category, const char *locale);
int main()
{
  if (!setlocale(LC_CTYPE, "")) 
  {
    fprintf(stderr, "Locale not specified. Check LANG, LC_CTYPE, LC_ALL.
");
    return 1;
  }
```

C 语言库将会依次测试环境变量 LC_ALL、LC_CTYPE 和 LANG。其中第一个含值的环境变量将决定为 LC_CTYPE 类别装入哪种语言环境数据。语言环境数据分裂成独立的类别。值 LC_CTYPE 定义了字符编码，而 LC_COLLATE 定义了排序顺序。我们用 LANG 环境变量为所有类别设置缺省语言环境，但 LC_* 变量可以用来覆盖单个类别。

您可以用命令         `locale charmap`  查询当前语言环境中字符编码的名称。如果您从 LC_CTYPE 类别中成功选取了 UTF-8 语言环境，会输出 UTF-8。命令         `locale -m` 提供一张已安装的所有字符编码名称的列表。      

如果您使用专门的 C 语言库的多字节函数来完成所有外部字符编码和内部使用的 wchar_t 编码之间的转换，那么 C 语言库将承担责任，根据 LC_CTYPE 使用正确的编码方式。这甚至不需要程序被明确的编码成当前的多字节编码。

如果需要一个应用程序能明确的支持 UTF-8（或其它编码）转换方法而不用 libc 多字节函数，则应用程序必须确定是否需要激活 UTF-8 模式。带有 <langinfo.h> 库头文件的与 X/Open 兼容系统可以用如下代码：

##### 清单 10. 检测当前的语言环境是否使用了 UTF-8 编码

```cpp
BOOL utf8_mode = FALSE;
if( !  strcmp(nl_langinfo(CODESET), "UTF-8")
   utf8_mode = TRUE;
```

为检测当前语言环境是否使用了 UTF-8 编码。首先必须调用         `setlocale(LC_CTYPE, "")`  函数，依据环境变量设置语言环境。nl_langinfo(CODESET) 函数也是由         `locale charmap`  命令调用，从而查找当前语言环境指定的编码名称。      

另一种可以使用的方法是查询语言环境变量：

##### 清单 11. 查询语言环境变量

```cpp
char *s;
BOOL utf8_mode = FALSE;
if ((s = getenv("LC_ALL")) || (s = getenv("LC_CTYPE")) || (s = getenv ("LANG"))) 
{
   if (strstr(s, "UTF-8"))
      utf8_mode = TRUE;
}
```

这项测试假设 UTF-8 语言环境名称中有值“UTF-8”，但实际情况并不总是如此，所以应该使用         `nl_langinfo()` 方法。      

## 总结

 为支持世界上的所有语言，需要一种具有八位字节字符编码策略的字符编码系统，它的字符应多于 ASCII（一种使用无符号字节的扩展版本）的 2^8 = 256 个字符。Unicode 就是这样一种字符编码系统，它具有由 128 个三维组（带有由大量字符编码方案的方法支持的 94,140 个定义好的字符值）组成的四维编码空间，在 Linux 中更流行的字符编码方案是 Unicode 转换格式 UTF-8。

#### 相关主题

- 您可以参阅本文在 developerWorks 全球站点上的          [英文原文](http://www.ibm.com/developerworks/library/l-linuni.html?S_TACT=105AGX52&S_CMP=cn-a-l).        

- 请访问 Unicode 联盟的          [Unicode 主页](http://www.unicode.org/)，这里定义了 Unicode 字符之间的行为和关系，并为实现者提供了技术信息。        

- [国际标准组织（International Organization for Standardization，ISO）](http://www.iso.ch/iso/en/ISOOnline.frontpage)是一个由 140 个国家组成的全球性的国家标准社团联盟。        

- [ANSI](http://web.ansi.org/) 是个私有的、非营利组织，它管理并调整 U.S. 的志愿标准化以及一致性评价系统。        

- [ISO C99 Draft](http://www.ucalgary.ca/~bgwong/n869.pdf)（Acrobat PDF 格式，556 页），是新的 C 语言标准，来自 Calgary 大学 Ben 的 C 编程课程。        

- 请阅读 Roman Czyborra 的          [Unix 环境下的 Unicode](http://czyborra.com/)。        

- 请阅读          [IANA（Internet Assigned Numbers Authority）](http://www.iana.org)中的          [IANA Charset Registration Procedures](ftp://ftp.isi.edu/in-notes/rfc2278.txt)。        

- 请参阅 Virginia 大学图书馆 Robertson Media 中心的          [Unicode Music Symbols](http://www.lib.virginia.edu/dmmc/Music/UnicodeMusic/)。        

- 请看看          [graphic representation of the Roadmap to the BMP, Plane 0 of the UCS](http://www.egt.ie/standards/iso10646/bmp-roadmap-table.html)。这些表包含了由 0 号，也就是通用字符集（Universal Character Set，UCS）的基本多语言平面（Basic Multilingual Plane，BMP）实际大小的映射组成的。Everson Gunn Teoranta 是一个自 1990 年开办的支持少数民族语言团体的软件和出版公司，由 Michael Everson 和 Marion Gunn 共同建立。        

- 请浏览          [UTF-8 and Unicode FAQ for UNIX/Linux](http://www.cl.cam.ac.uk/~mgk25/unicode.html)，Markus Kuhn 的综合性的 one-stop 信息资源，关于您如何在 POSIX 系统（Linux，UNIX）使用 Unicode/UTF-8。        

- 请检查 Technology Appraisals Ltd 的          [Solution Given by the Universal Character Set](http://www.techapps.co.uk/ucs.html)，其中提供了独立的、高质量的有关电子商务系统、电子信息传递、XML、网络和 IT 安全的信息、教育和培训。        

- 请阅读 Mulberry Technologies, Inc 的          [Unicode presentation titled“10646 and All That”](http://www.mulberrytech.com/papers/unicode/sld001.htm)，一个专攻基于 SGML 和 XML 系统的电子出版物的咨询公司。        

- 请咨询 Linux 程序员手册上的          [UTF-8 ― an ASCII compatible multi-byte Unicode encoding](http://www.cl.cam.ac.uk/~mgk25/ucs/man-utf-8.html)。        

- 请阅读          [Unicode Standard Annex#15 Unicode Normalization Forms](http://www.unicode.org/unicode/reports/tr15/)，一篇描写了四种 Unicode 文本标准化格式规范的文档。有了这些格式，等价的（规范或是兼容的）文本将会有同样的二进制表式。当实现工具在标准化的格式中保留了一个字符串，可以确保有一个以二进制形式表现的独一无二的等价字符串。        

- 请阅读 man-pages.net 上的          [`mbstowcs`](http://man-pages.net/linux/man3/mbstowcs.3.html)，它把多字节字符串转换成了宽字符的字符串，man-pages.net 为 Linux 手册页面提供了永久的基于 Web 的归档文件。        

- 请阅读 Hewlett Packard 的开发者资源站点的 Linux 程序员手册上的          [` wcsrtombs`](http://devresource.hp.com/STKL/man/RH6.1/wcsrtombs_3.html)，它能将宽字符的字符串转化为多字节字符串。        

- 请阅读 MKS 工具箱文档中的          [`setlocale()`](http://www.mkssoftware.com/docs/man3/setlocale.3.asp)，它能改变或查询语言环境。MKS 软件公司是在 Windows 环境或混合 UNIX/Linux 和 Windows 环境中用于系统管理和开发的 Windows 自动化工具的领先供应商。        

- 请学习          [ IBM Classes for Unicode (ICU)](http://www.ibm.com/cgi-bin/click.cgi?url=oss.software.ibm.com/icu/&origin=l)，一个 C 语言和 C++ 语言库，它在许多平台上提供了健壮的和功能完善的 Unicode 支持。        

- 请参阅 IBM 的          [ “Introduction to Unicode”站点](http://www.ibm.com/cgi-bin/click.cgi?url=oss.software.ibm.com/icu/userguide/unicodeBasics.html)，这里深入涵盖了 Unicode 基础知识。        

- 在 IBM 的关于新兴技术的          

   *alphaWorks*站点          

  。请参阅：                     

  - [ UnicodeCompressor](http://www.ibm.com/cgi-bin/click.cgi?url=www.alphaworks.ibm.com/tech/unicodecompressor&origin=l)，这里提供了使用标准 Unicode 压缩方案的压缩和解压缩 Unicode 文本的工具            
  - [ Unicode Normalizer](http://www.ibm.com/cgi-bin/click.cgi?url=www.alphaworks.ibm.com/tech/unicodenormalizer&origin=l)，为实现快速排序和搜索将 Java 字符串对象转换为标准 Unicode 格式。            

- 请阅读 TW Burger 撰写的          [“Cyrillic in Unicode”](http://www.ibm.com/developerworks/library/u-cyr/?S_TACT=105AGX52&S_CMP=cn-a-l)和 Jim Melnick 撰写的          [“Multilingual forms in Unicode”](http://www.ibm.com/developerworks/library/os-mult.html?S_TACT=105AGX52&S_CMP=cn-a-l)，也在          *developerWorks*上。        

- 请在          *developerWorks*上浏览          [更多 Linux 参考资料](https://www.ibm.com/developerworks/cn/linux/index.html)。        
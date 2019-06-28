---
title: C++中构建字符串的完全指南
date: 2019-04-17 12:50:25
tags: C++ 
---
>翻译自https://www.fluentcpp.com/2017/12/19/build-strings-from-plain-string-up-to-boost-karma/

对于一个程序员来说，无论使用哪种语言，创建一个字符串听起来都是一件很基本的事情。但是事实上在C++中，根据你想要实现的复杂度，有很多种不同的方法去创建一个字符串。下面就让我们看看一系列的实现方法，从最基本的标准库中的std::string到boost karma，在简洁的代码中领略复杂的字符串创建。

鉴于文章较长，下面是简要目录：

- 通过string来构建一个... 一个string
- 通过两个string来构建一个string
- 通过N个string来构建一个string
- 通过一个文件来构建一个string
- 扔掉除了string的所有东西
- Boost Format：从内容中按照格式分离
- Boost Karma，there we are

## 通过string来构建一个... 一个string

最基本的方法来构建一个字符串，相信你肯定会知道：
```cpp
    std::string greeting = "Hello, world";
```
### 结构化字符串

这一个方法知道的人就不多了—一个长字符串可以分割成若干行，而且只需要引号就行了，不需要别的特殊语法。
```cpp
std::string longGreetings = "Hello, world. How are you doing? I suppose that by now "
                                "you must have your inbox chock-full of greetings like "
                                "this one, in like hundreds of programming languages and "
                                "sent over by thousands or millions of software developers "
                                "taking up the challenge of learning a new language. "
                                "World, you must be the most popular mentor for beginners "
                                "but you'll find this message a little bit different: in "
                                "it you'll hear about Boost Karma, which I hope you'll "
                                "find both unusual and interesting. Keep it up, world.";
```

是不是很方便？

这个方法在写SQL数据库请求中还是很方便的，因为在有些时候这样的方式会让你的代码的可读性更好。哦，对了。不要忘记在每一行的结尾加上空格符，不然行末的单词就和下一行行首的单词拼在一起了。

这个技巧能你字符串构建的代码整齐有序而不是之前在笔直的一句代码中完成。
```cpp
    std::string s = "(field1=value1) or ((field6=value2 or field2=value3 or field3=value4) and (field1=value2))";
```
可以写成这样的形式：
```cpp
    std::string s = "("
                        "field1=value1"
                    ")"
                    " or "
                    "("
                        "("
                            "field6=value2"
                            " or "
                            "field2=value3"
                            " or "
                            "field3=value4"
                        ")"
                        " and "
                        "("
                            "field1=value2"
                        ")"
                    ")";
```
### 原生字符串

在代码中，字符串的结束符是一个引号"。但是想在字符串中包含一个引号，此时就需要转义符号了。
```cpp
    std::string stringInQuote = "This is a \"string\"";
    // output
    // This is a "string"
```
在C++11中，有个标识符raw string能让你把任意字符直接添加到字符串中。一个字符R表明该字符串是raw string，这个字符串的内容也要被包围起来，包围方法见下：
```cpp
    std::string stringInQuote = R"(This is a "string")";
    //这个字符和上方的完全一样
```
每一个包含在raw string的字符都会被构建到字符串中去，包括IDE中的换行符和空格。若代码如下：
```cpp
    std::string stringInQuote = R"(This is a "string"
                                   and a second line)";
```
其输出为：
```cpp
    This is a "string"
                                   and a second line
```
所以在使用raw string时一定要特别注意。如果你想要若干行的字符组成一个raw string，那么一定把每一行字符的缩进给去掉：
```cpp
    std::string stringInQuote = R"(This is a "string"
    and a second line
    and a third)";
```
### std::string的构造函数

本节最后一件事说的是std::string的构造函数。你可以用构造函数构建一个包含n个重复字符的字符串：

```cpp
    std::string s(10, 'a'); // read: 10 times 'a'
    std::cout << s << '\n';
    //outputs:
    //aaaaaaaaaa
```

## 通过两个string来构建一个string

在C++中最简单的拼接字符串的方法是使用+或者是+=运算符
```cpp
    std::string s1 = "Hello, ";
    std::string s2 = "world.";
     
    std::string s3 = s1 + s2;
    s1 += s2;
```
这两个运算符能接受几种不同的对象，包括指向字符串的const char*，甚至是单独的一个字符。
```cpp
    std::string s1 = "Hello, ";
    std::string s2 = s1 + "world.";
    //or even individual characters：
    s2 += '!';
```
现在你可能会好奇这两个运算符的运算性能，想知道使用operator+ 或者 operator+=真的是个好选择吗？我在Google Benchmark上做了一个对比测试，通过构建单个string来检测下面两段代码的性能区别：
```cpp
    //code 1
    std::string s4;
    s4 = s1 + s2 + s3;

    //code 2
    std::string s4;
    s4 += s1;
    s4 += s2;
    s4 += s3;
```
从结果来看，对于长字符串来说，这两者之间并没有显著的区别，operator+=在短字符串的操作上要更快一些。我怀疑其中[返回值优化](https://www.fluentcpp.com/2016/11/28/return-value-optimizations/)起到了一定的作用。该优化在不同的编译器下表现并不一样，所以你要是想要确定他们在你平台上真正的表现，最好还是自己做一下测试。

## 通过N个string来构建一个string

试想以下情景：你有一大堆的字符串，并且你想把它们组合成一个大字符串，在C++中你会怎么做呢？

一种能在一行代码中解决该问题的方法是使用 `std::accumulate`:
```cpp
    std::string result = std::accumulate(begin(words), end(words), std::string())
```
事实上，`std::accumulate`接收一个集合和一个初始值，不断的对集合中的元素和初始值调用+运算符，每次都用他们相加的结果来更细初始值。

注意⚠️，在这里初始值被设为`std::string()`而不是简单的“”，因为`std::accumulate`是把传进来的值作为一个模版参数。对于“”来说，在类型萃取的时候没有隐式转换，算法会认为其类型是`const char*`，这个类型和+运算符的返回值`std::string`是冲突的，这样accumulator就不能赋值了。

尽管这种方法十分的方便，但是速度却不是很快。在执行过程中，有很多临时的字符串被创建和销毁。因此，你可以尝试以下代码：
```cpp
    std::string result;
    for (std::string const& word : words)
    {
        result += word;
    }
```
利用Google benchmark进行的测试，第二种方法比第一种方法快了大概4.5倍。

## 通过一个文件来构建一个string

把一个文件中的内容读取出来并传到一个字符串中可以通过以下方式实现：
```cpp
    std::ostringstream fileContentStream;
    fileContenStream << std::ifstream("MyFile.txt").rdbuf();
    std::string fileContents = fileContentStream.str();
```
<img src="/image/TBD.jpeg">

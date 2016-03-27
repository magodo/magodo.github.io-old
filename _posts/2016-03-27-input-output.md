---
layout: "post"
title: "输入和输出(I/O)"
categories:
- "cpp"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

# 1. 输入输出流(I/O stream)

输入输出虽然不是C++核心语言的一部分，但是它由C++标准库提供。因此在`std`这个命名空间内。

## 1.1 iostream库

当你加入*iostream*这个头文件时，C++会提供你一套I/O相关的类结构（class hierarchy）:

![iostream class hierarchy](/images/cpp/iostream_class_hierarchy.png)

**stream**

抽象层面上，一个流代表一串可以被顺序访问的字符。随着时间的推移，一个流可以产生(produce)数据，也会消耗(consume)数据.

通常，我们会处理两类流：输入流 和 输出流

输入流用于保从文件，网络，设备（例如键盘）等data producer端传来的数据。这些数据存于输入流中，直到程序来读取。

输出流用于将程序中的数据向文件，网络，设备（例如打印机）等data consumer端传输。这些数据会存于输出流中，直到data consumer准备完毕之后，才会真正被传出。

程序员只需要学会如何与这些输入流或者输出流交互，就可以与各种不同的设备进行数据传输。而其中不同设备的实现细节，则交由操作系统去处理。

## 1.2 C++中的I/O

**istream** 是用于处理输入流的类。它主要提供了一个extraction operator(>>)，用于将数据从输入流中取出。其中的数据是由其他设备产生的。

**ostream** 是用于处理输出流的类。它主要提供了一个insertion operator(<<)，用于将数据存入输出流中。其中的数据将被其他设备消耗。

**iostream** 既可用于输入也可用于输出。

**_withassign** 的类多了一个assingment operator，允许用户将一个流赋值给另一个流。大部分情况下，不会用到这些类。

## 1.3 C++中的标准流

C++中有4种预定义的标准流：

* **cin** -- istream_withassign 类，绑定到标准输入（通常为键盘）
* **cout** -- ostream_withassign 类，绑定到标准输出（通常为显示器）
* **cerr** -- ostream_withassign 类，绑定到标准错误（通常为显示器）。提供非缓冲输出。
* **clog** -- ostream_withassign 类，绑定到标准错误（通常为显示器）。提供缓冲输出。

# 2. 使用istream输入

## 2.1 抽取操作符(>>)

`>>`可以用于从输入流中读取数据。在读取数据的时候，有个普遍的问题是，如何保证输入流中的数据不会溢出你的接收buffer。例如：

{%highlight CPP linenos%}
char buf[5];
std::cin >> buf;
{%endhighlight%}

如果用户通过键盘向标准输入流中输入超过5个字符(不包括换行符或EOF)，那么以上的代码就会溢出buffer。通常，我们不应该对输入流中数据的长度做任何假设。

有一种解决方法是使用**manipulator**. **manipulator**是一种与`<<`或`>>`一起使用时修改stream的对象。例如，`std::endl`就是一个**manipulator**，它既输出一个换行符，又flush任何被缓存的数据。

C++提供另一个manipulator -- **setw**(头文件：*iomanip*) -- 可被用于限制从流中读入字符的数量。使用setw，仅需传入限制个数，并且插入在流和buffer之间：

{%highlight CPP linenos%}
std::cin >> std::setw(5) >> buf;
{%endhighlight%}

这样，如果用户输入了6个字符，那么第一次只会读出其中的前4个字符留一位存EOF。第二次再调用`>>`会继续读入剩余的2位:

{%highlight CPP linenos%}
#include <iostream>
#include <iomanip>

int main()
{
    char buf[5];

    std::cin >> std::setw(5) >> buf;
    std::cout << "*\n" << buf << "\n*\n";
    std::cin >> std::setw(5) >> buf;
    std::cout << "*\n" << buf << "\n*\n";
}
{%endhighlight%}

运行：

    stream > ./a.out 
    123456
    *
    1234
    *
    *
    56
    *

## 2.2 cin的其他成员函数

1. `>>` 不会保存输入字符串中的空格（blanks, tab, newline）
2. `cin.get(char&)` 用于读取一个字符存于char变量中
3. `cin.get(char *buf, int limit)` 用于读取最多*limit-1*个字符存于buf中，然后在buf中补上EOF。
   不过，buf不会读取输入流中的换行符，并且在遇到换行符的时候就停止输入，返回。这会导致后续的对该输入流的`get`读取立即返回，因为第一个字符就是换行符。例如：

        #include <iostream>

        int main()
        {
            char buf[10];
            char ch;
            std::cin.get(buf, 10);
            std::cout << buf;

            std::cin.get(buf, 10);
            std::cout << buf;
        }

   运行:

        stream > ./a.out 
        hello
        hello02_istream > 

   可见，第二个`get`立即返回了。

4. `cin.getline(char *buf, int limit)` 和`get`的唯一区别就是会读取换行符，但是不会将其存入buf中。例如：

        #include <iostream>
        #include <string.h>

        int main()
        {
            char buf[10];
            char ch;
            std::cin.getline(buf, 10);
            std::cout << buf;
            std::cout << "Last read: " << std::cin.gcount() << " characters."  << '\n';
            std::cout << "Length of buf: " << strlen(buf) << '\n';

            std::cin.getline(buf, 10);
            std::cout << buf;
            std::cout << "Last read: " << std::cin.gcount() << " characters."  << '\n';
            std::cout << "Length of buf: " << strlen(buf) << '\n';
        }

   运行：

        02_istream > ./a.out                                                                             
        hello
        helloLast read: 6 characters.
        Length of buf: 5
        hi
        hiLast read: 3 characters.
        Length of buf: 2

5. `cin.gcount()` 返回上次被读取的字符个数。

6. `cin.ignore()` 丢弃stream中的第一个字符

7. `cin.ignore(int nCount)` 丢弃stream中的前nCount个字符

8. `cin.peek()` 允许从stream中读取一个字符并且依然将该字符保存于流中

9. `cin.unget()` 将上一个被读取出stream的字符重新返回stream中，从而会被下一个读取操作读取

10. `cin.putback(char ch)` 允许将任意字符放入stream中，从而会被下一个读取操作读取

# 3. 使用ostream输出

插入操作符`<<`用于将数据传入一个输出流。

## 3.1 格式化输出

输出数据支持格式化输出(formatting)。有两种方式可以改变输出的格式：**flag** 和 **manipulators**

### 3.1.1 Flag

Flag可以被认为是一个布尔类型变量，可以被打开或关闭。

* 打开某个选项，使用`setf()`函数。打开多个选项，可以使用`|`；
* 关闭某个选项，使用`unsetf()`函数

例如：

{%highlight CPP linenos%}
#include <iostream>

int main()
{
    std::cout.setf(std::ios::showpos); // 显示"+"，对于positive number
    std::cout << 27 << '\n';
}
{%endhighlight%}

输出： `+27`

有一个需要注意的是：有些flag属于某些groups，称为**format group**. 一个**format group**中的flag通常提供类似的格式化功能，但往往是互斥的。例如，有一个**format group**称为*basefield*，它包含*oct*,*dec*,*hex*三个flag，用于控制整数的进制。默认情况下"dec"选项是打开的。按理说，如果我们想改变进制为16进制，可能会写成：

{%highlight CPP linenos%}
#include <iostream>
using namespace std;
int main()
{
    cout.setf(ios::hex);
    cout << 27 << '\n';
}
{%endhighlight%}

可是输出仍然为：`27`

原因是，`setf()`函数仅仅将某个flag打开 -- 它还没聪明到能够将互斥的开关关闭。因此，上例中ios::hex与ios::dec实际上都是打开状态的，而ios::dec显然有更高的优先级。

有两种方式可以解决这个问题：

1. 手动关闭ios::dec

        cout.unsetf(ios::dec);
        cout.setf(ios::hex);
        cout << 27 << endl;

2. 使用`setf()`函数的另一个形式，输入两个参数：第一个参数要被打开的flag，第二个参数是flag所处的formatting group。当使用这种形式的`setf()`时，所有属于该group的flag都被关闭，除了被传入的flag被打开。例如上例，我们可以：

        cout.setf(ios::hex, ios::basefield);
        cout << 27 << endl;

### 3.1.2 Manipulators

相对与FLag而言，Manipulator会好用很多。它是在stream输入输出语句中的一种对象，用于改变输入或输出的方式。

Manipulaotr的一个好处是，它会自动打开和关闭某些flag，例如上面改变整数进制的情况，使用Manipulator:

{%highlight CPP linenos%}
#include <iostream>

int main()
{
    using namespace std;
    cout << oct << 27 << endl;
    cout << 27 << endl;   // 依然是oct进制
    cout << dec << 27 << endl;
    cout << hex << 27 << endl;
}
{%endhighlight%}

输出：

{%highlight CPP linenos%}
33
33
27
1b
{%endhighlight%}

## 3.2 一些使用的formater(包括flag, manipulator, member function)

这里列出一些比较常用的flags, manipulators 以及 member functions。其中，

* flags属于`ios`类;
* manipulators属于`std` namespace;
* member function属于`ostream`类。

### 3.2.1 布尔显示

|---
| Group | Flag | Meaning |
|:-|:-|:-|
|| boolalpha | 如果打开，bool值输出为"true" or "false";否则，输出为 0 or 1
|===

|---
|Manipulator|Meaning
|:-|:-|
|boolalpha|布尔值输出为"true" or "false"
|noboolalpha|布尔值输出为 0 or 1
|===

### 3.2.2 正数"+"号显示


|---
| Group | Flag | Meaning |
|:-|:-|:-|
|| showpos | 如果打开，则在正数前加上"+"
|===

|---
|Manipulator|Meaning
|:-|:-|
|showpos|在正数前加"+"
|noshowpos|不在正数前加"+"
|===

### 3.2.3 大小写

|---
| Group | Flag | Meaning |
|:-|:-|:-|
|| uppercase | 如果打开，则显示大写的字母(例如：1.23456E+007)
|===


|---
|Manipulator|Meaning
|:-|:-|
|uppercase|显示大写的字母
|nouppercase|显示小写字母
|===

### 3.2.4 整数的进制


|---
| Group | Flag | Meaning |
|:-|:-|:-|
|basefield| dec | 十进制显示整数
|basefield| hex | 十六进制显示整数
|basefield| oct | 八进制显示整数
|basefield| (none) | 按照整数开头的字符来决定(0,none,0x)
|===


|---
|Manipulator|Meaning
|:-|:-|
|dec|十进制显示整数
|hex|十六进制显示整数
|oct|八进制显示整数
|===

### 3.2.5 浮点数的精度，记号法(notation)，小数点

|---
| Group | Flag | Meaning |
|:-|:-|:-|
|floatfield|fixed|使用decimal的方式显示浮点数
|floatfield|scientific|使用科学技术发显示浮点数
|floatfield|(none)|对于小数短的情况，使用fixed;否则，使用科学技术法
|floatfield|showpoint|总是显示小数点和后面的0

|---
|Manipulator|Meaning
|:-|:-|
|fixed|使用decimal的方式显示浮点数
|scientific|使用科学技术发显示浮点数
|showpoint|总是显示小数点和后面的0
|noshowpoint|只显示有效数，如果小数点后没有0则不显示小数点
|setprecision(int)|
|===

|---
|Member function|Meaning
|:-|:-|
|precision()|返回当前的精度
|precision(int)|设置精度并且返回之前的精度
|===

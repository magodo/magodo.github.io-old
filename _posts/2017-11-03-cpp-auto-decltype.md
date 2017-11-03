---
layout: "post"
title: "C++11 auto && decltype"
categories:
- "cpp"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

# 1. auto

auto的引入可以让编译器自动推断某个变量的类型，减少程序员的输入。例如以下代码：

    vector<int> v;

    for (vector<int>::iterator it = v.begin(); it != v.end(); ++it)
    { *it >> cout; }

有了`auto`后可以写成:

    vector<int> v;

    for (auto it = v.begin(); it != v.end(); ++it)
    { *it >> cout; }

因为，编译器是可以根据`v.begin()`来推断出`it`的类型，而不需要用户手动指定。

## 1.1 auto 的推断过程

考虑以下例子：

    int x = int{};
    const int& crx = x;

    auto something = crx;

`something`的类型并不是`const int&`, 而是`int`. 因为`auto`通过以下步骤推断一个表达式的类型：

1. 如果表达式是一个引用，引用被去掉；
2. 如果在第1步以后，有**top-level**的 `const` and/or `volatile`, 也被去掉。

注意第二步里的**top-level**, 这往往是从右往作读(top->low)，例如：

    int i{};
    const int *ci1 = &i; // has low-level const, don't ignore
    int *const ci2 = &i; // has top-level const

    auto something1 = ci1; // const int*
    auto something2 = ci2; // int*

另外，以上的这个推断的过程和模板函数对输入参数的推断过程是一样的，除了：`auto`可以额外推断`std::initializer_list`类型，而模板函数不可以。

所以，同样地情况在函数推断中：

    template<class T>
    void foo(T arg);

    foo(crx);

`crx`的类型也被推导为`int`. 

注意，以上两条是对于没有修饰符时`auto`的推断过程。

## 1.2 有修饰符的auto的另类推断过程

在上面的模板函数的例子中，如果我们希望实际的输入函数的类型被推导为`const int&`. 有以下两种写法：

    // 在调用的时候显示指出
    foo<const int&>(crx);

    // 或者在声明模板函数时指出
    template<class T>
    void foo(const T& arg);

后者同样可以用于`auto`，通过加入修饰符来指定推断出的类型是`const T&`:

    const auto& something = crx;

于是，`something`这次被推断为`const auto&`.

接下来需要注意，加入修饰符之后的`auto`的推断过程有两点需要注意:

1. `auto&`，表达式为`const`

    例如：

        const int c = 0;
        auto& ac = c;

    这个例子中的`ac`的类型是`const int&`。这和之前说的第二步中的去掉const不同，如果这里把const丢掉的话，`ac`就可以修改一个`const`变量了，因此语言设计时这种情况中`const`被保留了。

2. `auto&&`，表达式为lvalue or rvalue

    例如：

        int i = 123;
        auto&& ri_1 = i;    // lvalue
        auto&& ri_2 = 123;  // (p)rvalue

    这种情况下的推断过程如下：

    * 如果表达式是lvalue, 首先进行正常的推断过程(去掉reference和const/volatile)，然后在最终的类型上加上一个引用(&);
    * 如果表达式是rvalue，仅进行正常的推导过程。
    
    因此，对于以上两个例子：

    1. auto&& ri_1 = i;

        首先，进行无修饰符的推导，得到的结果是`int`，然后加上一个引用，算上指定的rvalue reference，我们得到了: `auto& &&`. 根据*reference collapsing rules*, `& &&` -> `&`. 所以，结果是`int&`.

    2. auto&& ri_2 = 123;

        首先，进行无修饰符推导，得到`auto`的结果`int`. 因此，最终的结果是`int&&`. 如果想知道这里这么推导的原因，可以查询*universal referene*或者*forwarding reference*.

## 1.3 设计理念

`auto`之所以丢掉`&`和`top-level const/volatile`，是因为即使有一个引用来推断变量的类型，不代表我们实际要的类型是引用（大概率并不是），因此不要`&`; `top-level const`也是同理。
 
# 引用

[1] [thbecker: "auto and decltype"](http://thbecker.net/articles/auto_and_decltype/)


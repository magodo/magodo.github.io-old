---
layout: "post"
title: "C++: 操作符重载"
categories:
- "cpp"
---

<!--more-->

C++的操作符重载有以下几个注意点：

1. 至少有一个操作数是用户自定义的类型（类）；
2. 只能重载C++中已经存在的操作符（例如不能重载`**`）；
3. 所有被重载的操作符保持它们之前的**优先级**和**结合方向**

C++的操作符重载一般可以通过两种方法实现：

* 通过`friend function`实现
* 通过`class member function`实现


# 1. 使用 `friend function` 进行操作符重载

使用`friend function`进行操作符重载一般用于那些不会对该类型内部变量进行修改的情况。

(虽然只要在定义该函数的时候，使用"non-const"引用来声明参数也是可以修改内部变量。可是，这样子的做法违反了封装原则，即类型的私有变量会被外界函数修改)

## 1.1 重载算术操作符（+，-，*，/）

由于算术操作符不会改变操作数，因此只需要使用`friend function`来重载即可。

{% highlight CPP linenos %}

#include <iostream>

using namespace std;

class Playground
{
    private:
        int m_length;
    public:
        Playground(int len);
        int getLength();

        // 操作数都为自定义类型
        friend Playground operator+(const Playground &obj1, const Playground &obj2);

        // 操作数中其中一个为自定义类型，每个操作需要定义两个
        friend Playground operator+(const Playground &obj, int len);
        friend Playground operator+(int len, const Playground &obj);
};

Playground::Playground(int len)
{
    m_length = len;
}

int Playground::getLength()
{
    return m_length;
}

Playground operator+(const Playground &obj1, const Playground &obj2)
{
    // 由于是friend function, 因此可以访问该类的对象的private成员
    return Playground(obj1.m_length + obj2.m_length);
}

Playground operator+(const Playground &obj, int len)
{
    // 由于是friend function, 因此可以访问该类的对象的private成员
    return Playground(obj.m_length + len);
}

Playground operator+(int len, const Playground &obj)
{
    // 这里直接使用了上面重加载的函数，可以使代码紧凑和简洁
    return obj + len;
}

int main()
{
    Playground obj1(1), obj2(2);
    Playground sum1 = obj1 + obj2;
    Playground sum2 = obj1 + 10;
    Playground sum3 = 20 + obj1;
    cout << sum1.getLength() << endl;
    cout << sum2.getLength() << endl;
    cout << sum3.getLength() << endl;
}

{% endhighlight %}

几个注意点：

1. 由于重载函数是`friend function`, 因此可以直接在函数内访问/修改（如果输入参数为non-const）该类的对象的private成员；
2. 当操作符不会改变操作数时（例如所有算术操作符都不会），则把`friend function`中的操作数都定义为`const`；
3. 当只有一个操作数是该类型，需要定义两个`friend function`来处理不同的参数顺序。其中一个函数的实现可以借助于调用另一个。

## 1.2 重载I/O操作符

有时候，自定义的类中有很多状态，想要一次性输出所有状态到输出流中需要很多trival的操作。这时候，可以通过重载自定义类型的`<<`操作符来达到你想要定义的输出格式。

重载`<<`时需要注意的是，`<<`的输入参数分别为`ostream`与自定义类（如：`std::cout << MyClass`）

又有时候，自定义类的输入方式也想改变，这时候可以通过重载其`>>`操作符。

重载`>>`时需要注意的是，`>>`的输入参数分别为`istream`与自定义类（如：`std::cin >> MyClass`）；还需要注意的是，`>>`一般会需要改变对象的private成员哦，这样子的话，`friend function`中的第二个参数必须为`non-const`!

此外，这两个操作符返回的都是`istream`/`ostream`的引用，这样可以支持chainable operation!


{% highlight cpp linenos %}

#include <iostream>

class MyData
{
    private:
        int m_year;
        int m_month;
        int m_day;

    public:
        MyData(int year = 2016, int month = 3, int day = 6);

        /* 注意返回值是ostream/istream的引用！！！ 
           返回ostream/istream是为了支持"chain"操作；
           返回对其的引用可别忘了。。
        */
        friend std::ostream& operator<<(std::ostream&, const MyData&);
        friend std::istream& operator>>(std::istream&, MyData&);
};

MyData::MyData(int year, int month, int day)
{
    m_year = year;
    m_month = month;
    m_day = day;
}

/* 注意：这里第二个操作数可以是引用也可以不是。个人认为不作为引用比较好，
         因为有时候要输出的对象是anonymous object，是个rvalue，没有引用！*/
std::ostream& operator<<(std::ostream &out, const MyData &obj)
{
    out << "-------\nDate Info\n-------\n"
        << "Year: " << obj.m_year << "\n"
        << "Month: " << obj.m_month << "\n"
        << "Day: " << obj.m_day << "\n";
    return out;
}

/* 注意， >> 的第二个参数不再是const的了，因为要对其成员进行修改！ */
std::istream& operator>>(std::istream &in, MyData &newday)
{
    std::cout << "Enter new day(year month day):\n";
    in >> newday.m_year;
    in >> newday.m_month;
    in >> newday.m_day;
    return in;
}

int main()
{
    MyData today, newday;

    std::cin >> newday;
    /* chainable 操作 */
    std::cout << newday << today;
}
{% endhighlight %}

以上代码输出：

{%highlight CPP linenos%}

➜  io_operator ./a.out  
Enter new day(year month day):
2016
10
2
-------
Date Info
-------
Year: 2016
Month: 10
Day: 2
-------
Date Info
-------
Year: 2016
Month: 3
Day: 6

{%endhighlight%}

## 1.3 重载比较操作符（==, !=, <, <=, >, >=）

比较操作符的重载和算术操作符类似，因为它们都不会修改操作数。所以，我们依然使用`friend function`，传入`const`参数。

{%highlight CPP linenos%}

#include <iostream>

class Score
{
    private:
        /* 假设成绩的比较有优先级：
           编程 > 数学
         */
        int m_programming;
        int m_math;

    public:
        Score(int prog = 0, int math = 0);
        friend bool operator==(const Score&, const Score&);
        friend bool operator!=(const Score&, const Score&);
        friend bool operator>(const Score&, const Score&);
        friend bool operator>=(const Score&, const Score&);
        friend bool operator<(const Score&, const Score&);
        friend bool operator<=(const Score&, const Score&);
};

Score::Score(int prog, int math)
{
    m_programming = prog;
    m_math = math;
}

bool operator==(const Score &score1, const Score &score2)
{
    return (score1.m_math == score2.m_math) && (score1.m_programming == score2.m_programming);
}

bool operator!=(const Score &score1, const Score &score2)
{
    /* 利用 == */
    return !(score1 == score2);
}

bool operator>(const Score &score1, const Score &score2)
{
    if (score1.m_programming == score2.m_programming)
        return (score1.m_math > score2.m_math);
    else
        return (score1.m_programming > score2.m_programming);
}

bool operator<=(const Score &score1, const Score &score2)
{
    /* 利用 > */
    return !(score1 > score2);
}

bool operator<(const Score &score1, const Score &score2)
{
    if (score1.m_programming == score2.m_programming)
        return (score1.m_math < score2.m_math);
    else
        return (score1.m_programming < score2.m_programming);
}
bool operator>=(const Score &score1, const Score &score2)
{
    /* 利用 < */
    return !(score1 < score2);
}

int main()
{
    using namespace std;

    Score kinokoScore(100, 100), magodoScore(100, 0);
    if (kinokoScore != magodoScore)
    {
        cout << "Not equal!\n";
        if (kinokoScore > magodoScore)
            cout << "Magodo is stupid > ~ <\n";
        else
            cout << "Kinoko is stupid > ~ <\n";
    }
    else
        cout << "Equal!\n";
}
{%endhighlight%}

以上代码输出：

{%highlight CPP linenos%}
➜  comparison_operator ./a.out 
Not equal!
Magodo is stupid > ~ <
{%endhighlight%}

## 1.4 重载一元操作符（+, -, !）

首先，一元操作符和之前所说的算术操作符和比较操作符一样，不会改变操作数，所以，我们依然使用`friend function`，传入`const`参数。

其次，一元操作符只有一个输入参数！

{%highlight CPP linenos%}
#include <iostream>

class Point
{
    private:
        int m_x;
        int m_y;

    public:
        Point(int x = 0, int y = 0);
        friend Point operator-(const Point &point);
        friend bool operator!(const Point &point);
        friend std::ostream& operator<<(std::ostream &out, Point point);
};

Point::Point(int x, int y)
{
    m_x = x;
    m_y = y;
}

Point operator-(const Point &point)
{
    return Point(-(point.m_x), -(point.m_y));
}

bool operator!(const Point &point)
{
    return (point.m_x == 0 && point.m_y == 0);
}

/* 注意： 这里不同与之前重载IO那里第二个输入参数为引用，这里第二个输入参数为类的对象。
          之所以这样设置的原因是，如果当要输出的对象只是个anonymous object, 也就意味
          着该对象只是一个rvalue（例如这里"-"返回的对象）, 是没有引用的！*/
std::ostream& operator<<(std::ostream &out, Point point)
{
    out << " (" << point.m_x << ", " << point.m_y << ") ";
    return out;
}

int main()
{
    Point origin_point, some_point(3, 3), OP = -some_point;

    if (!origin_point)
    {
        std::cout << origin_point << "is origin point!\n";
    }
    if (!some_point)
    {
        std::cout << some_point << "is origin point!\n";
    }

    std::cout << "Oposite " << some_point << " -> " << -some_point << '\n';
}
{%endhighlight%}

注意, 这里对于`<<`的重载不同与之前重载IO那里第二个输入参数作为类对象的引用，这里第二个输入参数直接是类的对象。之所以这样设置的原因是，如果当要输出的对象只是个anonymous object, 也就意味着该对象只是一个rvalue（例如这里"-"返回的对象）, 是没有引用的！

以上代码输出：

{%highlight CPP linenos%}
➜  unary_operator ./a.out  
 (0, 0) is origin point!
 Oposite  (3, 3)  ->  (-3, -3)
{%endhighlight%}

# 2. 使用成员函数重载操作符

使用成员函数进行操作符重载一般用于那些会修改类的内部变量的操作符。

这种操作符的重载方式有以下限制：

* 最左边的操作数必须是该类的一个对象；
* 最左边的操作数是该成员函数隐式传入的`this`指针，其他的操作数则是该函数的参数

大部分的操作符既可以通过该方式被重载，也可以通过`friend function`来重载 ，除了以下的例外：

* 以下操作符需要通过`friend function`进行重载：
  * `operator+(int, YourClass)`
  * `oeprator<<(ostream&, YourClass)`
  * `oeprator>>(istream&, YourClass)`
  * 等等

  因为，这些操作符最左边的操作数不是该类的一个对象。

* 以下操作符需要通过成员函数进行重载：
  * assignment (=, +=, -=, *=, /=, ++, --, ...)
  * subscript ([])
  * call (())
  * member selection (->)

## 2.1 使用member function 实现某些操作符的重载


{%highlight CPP linenos%}
#include <iostream>

class Point
{
    private:
        int m_x;
        int m_y;

    public:
        Point(int x = 0, int y = 0);

        /* 成员函数 （注意，第一个参数是隐式传入的this指针） */
        Point operator+(const Point&);

        /* 成员函数 （注意，第一个参数是隐式传入的this指针） */
        Point operator-();
        
        /* 只能使用friend function */
        friend std::ostream& operator<<(std::ostream&, const Point);
};

Point::Point(int x, int y)
{
    m_x = x;
    m_y = y;
}

Point Point::operator+(const Point &point)
{
    return Point(m_x+point.m_x, m_y+point.m_y);
}

Point Point::operator-()
{
    return Point(-m_x, -m_y);
}

std::ostream& operator<<(std::ostream &cout, const Point point)
{
    cout << "(" << point.m_x << ", " << point.m_y << ") ";
}

int main()
{
    Point point(3, 3);

    std::cout << -point << std::endl;
    std::cout << point + point << std::endl;

}
{%endhighlight%}
 
输出：

{%highlight CPP linenos%}
➜  intro ./a.out 
(-3, -3) 
(6, 6) 
{%endhighlight%}

注意，C++看到成员函数的原型（例如：`Point Point::operator-()`）时，编译器会将其转化为帶`this`指针的函数（例如：`Point operator-(const Point *this)`）。

## 2.2 重载 ++ 和 --

别忘了`++`和`--`都有两个版本哦！由于它们会改变类对象内部的成员变量，因此最好使用成员函数的方式来重载。


{%highlight CPP linenos%}
#include <iostream>

class Month
{
    private:
        int m_month;

    public:
        Month(int m = 1);
        Month& operator++();
        Month& operator--();
        Month operator++(int);
        Month operator--(int);
        friend std::ostream& operator<<(std::ostream&, const Month);
};

Month::Month(int m)
{
    m_month = m;
}

/* ++month */
Month& Month::operator++()
{
    if (m_month == 12)
    {
        m_month = 1;
    }
    return *this;
}

/* --month */
Month& Month::operator--()
{
    if (m_month == 1)
    {
        m_month = 12;
    }
    return *this;
}

/* month++ */
Month Month::operator++(int)
{
    Month temp(m_month);
    ++(*this);
    return temp;
}

/* month-- */
Month Month::operator--(int)
{
    Month temp(m_month);
    --(*this);
    return temp;
}

std::ostream& operator<<(std::ostream &out, const Month month)
{
    out << month.m_month;
    return out;
}

int main()
{
    Month month(12);
    std::cout << ++month << '\n';
    std::cout << --month << '\n';

    std::cout << month++ << '\n';
    std::cout << month << '\n';
    std::cout << month-- << '\n';
    std::cout << month << '\n';
}
{%endhighlight%}

输出：


{%highlight CPP linenos%}
➜  inc_dec ./a.out       
1
12
12
1
1
12
{%endhighlight%}

注意：

1. 由于C++在函数重载的过程中是通过函数的参数的数量和类型来区分重载的方式的，对于`postfix`形式的`++`/`--`，函数参数的数量和类型与`prefix`形式的是相同的。C++提供一种叫做`dummy variable`或者`dummy argument`的方式来进行区分，利用一个假的(fake)整形(`int`)参数来作为`postfix`与`prefix`的区分（我们甚至在定义函数的时候都不许要给这个int参数一个名字！）；
2. `prefix`形式，我们返回的是该对象自己的一个引用；`postfix`形式，我们返回的是一个新的object，其内容为调用对象在调用前的内容；
3. 我们在定义`postfix`形式的时候，直接使用之前已经定义过的`prefix`形式的代码。减少代码量并且降低出错的几率。

## 2.3 重载下标符(subscriptor) []


{%highlight CPP linenos%}
#include <iostream>
#include <cassert>

class MyString
{
    private:
        char m_string[10];

    public:
        MyString();

        /* 注意，返回值为引用哦 */
        char& operator[](int);
};

MyString::MyString()
{
    for (int i = 0; i < 10; i++)
        m_string[i] = 'a';
}

char& MyString::operator[](int subscript)
{
    /* 这里可以进行边界检查 */
    assert(subscript < 10 && subscript >= 0);
    return m_string[subscript];
}

int main()
{
    MyString mystring;

    mystring[0] = 'b';
    std::cout << mystring[0] << '\n';
    std::cout << mystring[1] << '\n';
}
{%endhighlight%}

输出：

{%highlight CPP linenos%}
➜  subscript ./a.out  
b
a
{%endhighlight%}

注意：

1. 由于下标操作符的返回值应该是个lvalue(存在内存空间)，而指针和引用都保证了这一点，因此，在重载subscript的时候需要将返回值声明为引用。这样，返回的m_string元素是真正的m_string元素，也就支持直接对其就行修改了；
2. 重载下标操作符可以使程序员进行边界检查，这是很容易出错的一个点，因为编译器不会报错，只有运行时才会暴露。

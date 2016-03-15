---
layout: "post"
title: "继承(inheritance)"
categories:
- "cpp"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

# 1. 基础

继承发生在类和类之间，被继承的类拥有其父类(或：基类，base class)的所有成员变量和成员函数，成为被继承类的一部分。

与组合（composition）加入了"has-a"的关系相比，继承类之间是"is-a"的关系。

# 2. 继承类的构造函数

## 2.1 构造函数调用顺序

如果是一条直线的继承，例如：

![linear inheritance](/images/cpp/linear-inherit.png)

则构造函数的顺序是： A->B->C. 

这是符合逻辑的，因为基类对子类的信息是不可知的，而子类却可以使用基类的信息去影响自己的构造过程。另一方面，子类是基于基类产生的。因此，基类的构造函数在子类之前被调用。

## 2.2 在子类的构造函数中初始化基类中的成员变量

假设基类和子类的声明如下：

{%highlight CPP linenos%}
class Base
{
public:
    int m_nValue;
 
    Base(int nValue=0)
        : m_nValue(nValue)
    {
    }
};
 
class Derived: public Base
{
public:
    double m_dValue;
 
    Derived(int nValue=0, double dValue=0.0);
};
{%endhighlight%}

由于子类继承了基类中所有的成员变量和成员函数，因此想要初始化基类中的成员变量(`m_nValue`)，可能有以下几种错误的做法：

1. 在子类的初始化列表(initialization list)中初始基类的成员变量:

        Derived::Derived(int nValue, double dValue)
                        :m_nValue(nValue)
                        ,m_dValue(dValue)
        {}

    这种做法在C++中是会报错的。C++不允许在初始化列表中初始化继承的成员变量。换句话说，C++的初始化列表中只允许初始化属于自己类的成员变量。

    C++之所以有这样子的设定是因为：当基类中的成员变量(`nValue`)是`const`或引用的时候，如果上述操作是允许的，那么就意味着子类有机会在其构造函数的初始化列表中去改变它的值！

    加上了这个限制之后，C++保证所有的成员变量只被初始化一次。

2. 在子类的构造函数的函数体内改变基类的值：

        Derived::Derived(int nValue, double dValue)
                        ,m_dValue(dValue)
        {
            m_nValue = nValue;
        }

    这样子的做法有两点缺点：

    * 当基类的成员变量是`const`或引用的话，这种方法不可行
    * 基类的成员变量被初始化了两次

正确的做法是：在子类的构造函数的初始化列表中显示地调用基类的构造函数

    Derived::Derived(int nValue, double dValue)
                    :Base(nValue)
                    ,m_dValue(dValue)
    {}

现在，假设实例化对象的时候，例如：`Derived object(1, 1.0);`. 实际的过程如下：

1. 为对象分配空间；
2. 调用`Derived`的构造函数；
3. 编译器检查我们有没有显示地去调用基类的构造函数：
    * 如果没有，则隐式地调用`Base`的默认构造函数
    * 如果有（上例），则调用之
4. `Base`的初始化列表被执行；
5. `Base`的构造函数函数体被执行；
6. `Derived`的初始化列表被执行；
7. `Derived`的构造函数的函数体被执行

Ju一个栗子:

{% highlight CPP linenos %}
#include <iostream> 
#include <string>

class Person
{
    public:
        std::string m_strName;
        int m_nAge;
        bool m_bIsMale;
        
        Person(std::string name = "", int age = 0, bool is_male = true);
        ~Person();
        std::string GetName();
        int GetAge();
        bool IsMale();
};

class FootballPlayer: public Person
{
    public:
        double m_dAverageGoal;
        double m_dAveragePassSuccessRatio;

        FootballPlayer(std::string name = "", int age = 0, bool is_male = true, double ave_goal = 0.0, double ave_pass_success_ratio = 0.0);
        ~FootballPlayer();
        double GetAverageGoal();
        double GetAveragePassSuccessRatio();
        void ShowStatData();
};

/* Member functions of Person */
Person::Person(std::string name, int age, bool is_male)
    :m_strName(name)
    ,m_nAge(age)
    ,m_bIsMale(is_male)
{
    std::cout << "Construction of Person\n";
}

Person::~Person()
{
    std::cout << "Deconstruction of Person\n";
}

std::string Person::GetName()
{
    return m_strName;
}

int Person::GetAge()
{
    return m_nAge;
}

bool Person::IsMale()
{
    return m_bIsMale;
}

/* Member functions of FootballPlayer */
FootballPlayer::FootballPlayer(std::string name, int age, bool is_male, double ave_goal, double ave_pass_success_ratio)
    :Person(name, age, is_male)
    ,m_dAverageGoal(ave_goal)
    ,m_dAveragePassSuccessRatio(ave_pass_success_ratio)
{
    std::cout << "Construction of FootballPlayer\n";
}

FootballPlayer::~FootballPlayer()
{
    std::cout << "Deconstruction of FootballPlayer\n";
}

double FootballPlayer::GetAverageGoal()
{
    return m_dAverageGoal;
}

double FootballPlayer::GetAveragePassSuccessRatio()
{
    return m_dAveragePassSuccessRatio;
}

void FootballPlayer::ShowStatData()
{
    using std::cout;
    cout << "Name                         : " << m_strName << '\n';
    cout << "Age                          : " << m_nAge << '\n';
    cout << "Gender                       : " << ((m_bIsMale)? "M":"F") << '\n';
    cout << "Average Goal                 : " << m_dAverageGoal << '\n';
    cout << "Average passing success ratio: " << m_dAveragePassSuccessRatio*100 << "%\n";
}

/* MAIN */
int main()
{
    FootballPlayer player1("magodo", 27, true, 0.5, 0.8);
    player1.ShowStatData();
}
{% endhighlight %}

输出：

{% highlight CPP linenos %}
➜  basic ./a.out 
Construction of Person
Construction of FootballPlayer
Name                         : magodo
Age                          : 27
Gender                       : M
Average Goal                 : 0.5
Average passing success ratio: 80%
Deconstruction of FootballPlayer
Deconstruction of Person
{% endhighlight %}

需要注意的是，子类只能调用其直接继承自的基类的构造函数，而不能调用基类的基类的构造函数。

# 3. 访问修饰符（public, private, protected）

访问修饰符，可以修饰类中的成员变量和成员函数；也可以修饰子类继承基类时的继承关系。

# 3.1 修饰类中的成员变量和函数

被访问修饰符修饰的类中的成员变量或函数的可访问情况如下表所示：

|---
||类本身|类的实例|类的子类|
|:-:|:-:|:-:|:-:|
|public|o|o|o|
|private|o|x|x|
|protected|o|x|o|
|===

<br>
其中： 

* o: 代表可以访问
* x: 代表不可易访问


例如：

{%highlight CPP linenos%}
class Base
{
    public:
        int m_nPublic;           // 能被 Base, Derived(public/protected继承), Base的对象访问
    protected:
        int m_nProtected;        // 能被 Base, Derieved(public/protected继承) 访问
    private:
        int m_nPrivate;          // 能被 Base 访问
};

class Derived: public Base
{
    public:
        Derived()
        {
            m_nPublic = 1;       // OK
            m_nProtected = 2;    // OK
            m_nPrivate = 3;      // NOK!
        }
}

int main()
{
    Base obj;

    obj.m_nPublic  = 1;          // OK
    obj.m_nProtected = 2;        // NOK!
    obj.m_nPrivate = 3;          // NOK!
}
{%endhighlight%}

# 3.2 修饰子类对于基类的继承关系

在继承基类的时候，可以选择是`public`, `private` 或者 `protected`的继承方式。如果不指定，C++默认是设为`private`的（和类中成员不指定的动作一样）。

* `public`: 基类中成员的修饰符在子类中原样保持；
* `protected`: 基类中成员的修饰符在子类中，`public`改为`protected`，其他原样保持；
* `private`: 基类中成员的修饰符在子类中都是`private`

假设有如下的继承关系：

A->B->C


则对B， B的对象，以及B的子类(C)的影响如下：

|---
|:-:|:-:|:-:|
||**B对A中变量的访问**|**B的实例对继承自A的变量的访问**|**C对B继承自A的变量的访问**|
|**public**|无影响|无影响|无影响|
|**protected**|无影响|可访问A中的public变量->不能访问|无影响|
|**private**|无影响|可访问A中的public变量->不能访问|可访问A中的public/protected变量->不能访问|
|===


  和对B的影响相同

实际上，对于一个特定的类，只要记住三点：

1. 类对于自己定义的成员，总是具有访问权限；
2. 子类只能访问它的直属基类，访问其`public`与`protected`的成员；
3. 类的对象只能访问该类的`public`的成员。如果该成员继承自基类，则要依据继承关系确定该成员在这个类中是否为`public`。






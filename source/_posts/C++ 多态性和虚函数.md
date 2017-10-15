---
title: C++ 多态性和虚函数
date: 2017-10-15
categories: C++
---

面向对象又一大特征
<!--more-->
# 一. 什么是多态
在 C++ 中, 发生继承关系之后, 子类对象赋给父类指针或引用, 通过父类指针或者引用调用某个方法, 根据子类对象的不同而体现出不同的效果.

# 二. 怎么实现多态
- 子类继承父类
- 父类中定义的函数是虚函数
- 将子类对象赋给父类指针或者引用
- 通过父类指针或引用调用函数

# 三. 什么是虚函数

## - 格式
```c++
virtual <返回值类型> <函数名>(<形式参数表>)
{
    <函数体>
}
```
## - 作用
虚函数是实现多态的基础
*如果不使用虚函数, 那么就算父类指针或者引用接收了子类对象, 当调用某个方法的时候, 调用的还是父类中定义那个函数.*

反面例子
```c++
#include <iostream>

const double PI = 3.14;

using namespace std;

/*
2017年10月14日 11:23:36
静态联编的问题
输出结果:
    Figue 的面积是:0
    Circle 的面积是:0
    Rectangle 的面积是:0
思路:
    子类都属于父类, 通过调用把子类的对象赋给父类的引用, 然后由父类的引用调用函数实现多态.
    但是这样是错误的, 这样调用的仍然是父类中的函数.
*/

class Figue
{
public:
    Figue(){};
    double area() const {return 0.0;}
};


class Circle: public Figue
{
public:
    Circle(double myr) {R = myr;}
    double area() const {return PI*R*R;} 
protected:
    double R;
};


class Rectangle: public Figue
{
public:
    Rectangle(double myl, double myw){L = myl; W = myw;}
    double area() const {return L*W;}
private:
    double L, W;
};

void func(Figue &p)
{
    cout<<p.area()<<endl;
}

int main()
{
    Figue fig;
    cout<<"Figue 的面积是:";
    func(fig);
    Circle c(3.0);
    cout<<"Circle 的面积是:";
    func(c);
    Rectangle r(4.0, 5.0);
    cout<<"Rectangle 的面积是:";
    func(r);

    return 0;
}
```

正面例子
```c++

#include <iostream>

const double PI = 3.14;

/*
2017年10月14日 11:23:21
动态联编的例子
输出结果:
    Figue 的面积是:0
    Circle 的面积是:28.26
    Rectangle 的面积是:20
基类中的 area 函数定义为虚函数
子类中覆盖(为了区分重载函数, 所以称之为覆盖)了虚函数
然后把子类对象赋给父类引用, 通过父类引用(通过指针也行)调用 area 实现调用不同子类的 area 方法
(多态就是通过父类调用方法体现出来的???)
*/

using namespace std;

class Figue
{
public:
    Figue(){};
    virtual double area() {return 0.0;}//定义为虚函数
};


class Circle: public Figue
{
public:
    Circle(){};
    Circle(double myr) {R = myr;}//定义为虚函数
    virtual double area() {return PI*R*R;}
protected:
    double R;
};


class Rectangle: public Circle
{
public:
    Rectangle(double myl, double myw) {L = myl; W = myw;}
    virtual double area() {return L*W;}
private:
    double L, W;
};

void func(Figue &p)
{
    cout<<p.area()<<endl;
}

int main()
{
    Figue fig;
    cout<<"Figue 的面积是:";
    func(fig);
    Circle c(3.0);
    cout<<"Circle 的面积是:";
    func(c);
    Rectangle r(4.0, 5.0);
    cout<<"Rectangle 的面积是:";
    func(r);

    return 0;
}
```

## - 虚函数和一般重载函数的区别
- 重载函数要求名字相同而参数不同, 虚函数要求函数原型完全相同.
- 重载函数可以是成员函数或友元函数, 虚函数只能是非静态函数.
- 构造函数可以重载, 析构函数不能重载.而构造函数不能定义为虚函数, 而析构函数可以.
- 重载函数的调用是根据参数序列, 而虚函数的调用是根据对象的不同.
- 重载函数在编译时表现出多态性, 是静态联编; 而虚函数则在运行时表现出多态性, 是动态联编, 因此说动态联编的是 C++ 的精髓.

## - 特点
- 基类中定义的虚函数, 那么派生类中这个函数仍然是虚函数, 因为虚函数具有自动向下传给派生类的性质.
- 在派生类中可以重新定义虚函数, 要求和基类的虚函数的原型完全相同, 此时关键字 virtual 可以省略, 但为了可读性, 通常不省略.
- 为区分重载函数, 重定义虚函数叫做覆盖.
- 如果派生类与基类的虚函数**仅仅返回类型不同**, C++ 会认为使用了不恰当的虚函数, 因为只靠返回类型进行函数匹配是模糊的.
- 如果派生类中与基类中的虚函数**仅仅形参不同**, 那么会被认为是定义了另一个虚函数, 而不是重新定义基类中的虚函数, 此时派生类中有两个虚函数.
- 派生类把基类中的普通函数重新定义为虚函数, 对基类毫无影响.

# 三. 成员函数中调用虚函数
一个基类或派生类的成员函数中可以直接调用该类等级中的虚函数.
(在这个🌰中, 派生类中的func2 中调用了虚函数 func1, 虽然 func2 不是虚函数, 通过基类引用调用 func2, func2 调用的是派生类中的 func1. )
```c++
#include <iostream>

using namespace std;

/*
2017年10月14日 12:08:38
成员函数调用虚函数
*/

class Base
{
public:
    virtual void func1()
    {
        cout<<"这是父类的 func1 方法."<<endl;
    }
    void func2() {func1();}
};


class Subclass: public Base
{
public:
    virtual void func1()
    {
        cout<<"这是子类的 func1 方法"<<endl;
    }
};

int main()
{
    Subclass sc;
    // Base & bc = sc;
    // bc.func2();
    sc.func2();

    return 0;
}
```

# 四. 构造函数和析构函数中调用虚函数

构造函数和析构函数中调用的虚函数是自己的类中定义的虚函数(也就是说在当前类中第一次出现的), 如果自己的类中没有这个虚函数, 则调用基类中的虚函数, 但绝不是任何在派生类中重定义(覆盖)的虚函数.
```c++
#include <iostream>

using namespace std;

/*
2017年10月14日 12:22:20
构造函数和析构函数中调用虚函数
输出结果:
    这是父类的 func1 方法.
    Exit main
    这是父类的 func2 方法.
1. 在构造函数和析构函数中调用虚函数, 调用的是类中定义的虚函数
   如果没有定义, 就调用父类中的那个虚函数.需要注意的是, 如果当前类中覆盖了虚函数,是不会被调用的.
*/


class Base
{
public:
    Base(){func1();}
    virtual void  func1()
    {
        cout<<"这是父类的 func1 方法."<<endl;
    }
    virtual void  func2()
    {
        cout<<"这是父类的 func2 方法."<<endl;
    }
    ~Base(){func2();}
};


class Subclass: public Base
{
public:
    virtual void func1()
    {
        cout<<"这是子类的 func1 方法."<<endl;
    }
    virtual void  func2()
    {
        cout<<"这是子类的 func2 方法."<<endl;
    }
};

int main()
{
    Subclass sc;

    cout<<"Exit main"<<endl;

    return 0;
}
```

# 五. 纯虚函数和抽象类
## - 纯虚函数
如果在基类中还不确定一个虚函数具体能干嘛, 就可以将其定义为纯虚函数.
### 格式
```c++
virtual <返回值类型><函数名>(<形式参数表>) = 0;
```
注意, 这里 = 0 只是一种形式, 而不是说它的返回值等于 0 ;

### 🌰
```c++
#include <iostream>

const double PI = 3.14;

using namespace std;


/*
使用纯虚函数
一般形式:
    virtual <返回值类型><函数名>(<形式参数表>) = 0;
*/


class Figure
{
public:
    Figure(){};
    virtual double area() = 0;    //定义为纯虚函数
};


class Circle: public Figure
{
public:
    Circle(double myr){R = myr;}
    virtual double area()
    {
        return PI*R*R;
    }
protected:
    double R;
};


class Rectangle: public Figure
{
public:
    Rectangle(double myl, double myw)
    {
        L = myl;
        W = myw;
    }
    virtual double area()
    {
        return L * W;
    }
private:
    double L, W;
};


void func(Figure &p)
{
    cout<<p.area()<<endl;
}


int main()
{
    Circle c(3.0);
    cout<<"Circle 的面积是: ";
    func(c);

    Rectangle r(4.0, 5.0);
    cout<<"Rectangle 的面积是:";
    func(r);

    return 0;
}
```

## 抽象类
- 只要包含了纯虚函数的类就是抽象类.
- 抽象类不能看成是一种类型, 所以不能说明抽象类的对象, 不能作为返回值类型, 不能作为形式参数.
- 可以说明抽象类的指针或引用, 以支持运行时的多态性.
- 如果要直接调用抽象类中定义的纯虚函数, 必须使用完全限定名, 即使用带有作用域分辨符的完全限定函数名???
- 一个抽象类的派生类, 要么覆盖所有的纯虚函数, 使派生类不是抽象类, 要么仍然将其说明为纯虚函数, 否则将会报错.
- 成员函数中可以调用纯虚函数, 但是不能在构造函数和析构函数中调用一个纯虚函数.

# 六. 虚析构函数
## 定义
```c++
virtual ~<类名>()
```
- 析构函数没有返回值和参数, 所以只能有一个.
- C++ 标准不支持虚析构函数.
- 基类的析构函数是虚函数, 那么所有派生类的析构函数都是虚函数.

## 必要性
```c++
#include <iostream>

using namespace std;

/*
2017年10月14日 13:24:20
析构函数的必要性
如果不将析构函数设为虚函数, 那么 delete 的时候会根据指针的类型调用析构函数, 而不会考虑多态性
*/

class Base
{
public:
    Base(){};
    virtual ~Base()
    {
        cout<<"Base 的析构函数被调用."<<endl;
    }
};


class Subclass: public Base
{
public:
    Subclass()
    {
        ptr = new int;
    }
    ~Subclass()
    {
        cout<<"Subclass 的析构函数被调用."<<endl;
        delete ptr;
    }
private:
    int * ptr;
};

int main()
{
    Base * b = new Subclass;

    delete b;

    return 0;
}
```
如果不使用虚析构函数, delete b 的时候, 会根据 b 的类型调用 Base 的析构函数, 而不会调用 Subclass 的析构函数. 
如果使用虚析构函数, 先调用 Subclass 的析构函数, 在调用 Base 的析构函数, 所以虚析构函数是必要的.

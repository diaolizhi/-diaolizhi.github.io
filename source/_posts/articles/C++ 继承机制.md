---
title: C++ 继承机制
date: 2017-10-10
categories: C++
---

面向对象的一大特征
<!--more-->

# 一. 什么是继承
- 继承把一个原有的类的成员为己所用.
- 被继承的类叫做基类
- 新的类叫做派生类
- 继承有两种方式:
    + 单继承: 派生类只有一个基类
    + 多继承: 派生类有多个基类

# 二. 单继承的一般形式
```c++
class <派生类名>: <继承方式><基类名>
{
    public:
        <公有数据和函数>
    protected:
        <保护数据和函数>
    private:
        <私有数据和函数>
}
```
例如
```c++
class A
{
    public:
        seta(int x){
            a = x;
        }
    private:
        int a;
};

class B: public A
{

};
```

# 三. 派生类的访问控制
定义派生类的时候指定了<访问控制>, 访问控制包括:
- 公有继承
    + 基类的公有成员, 保护成员分别是派生类的公有成员, 保护成员
    + 基类的私有成员, 派生类无法访问.
- 私有继承
    + 基类的公有成员和保护成员都变成派生类的私有成员
    + 基类的私有成员, 派生类无法访问.
- 保护继承
    + 基类的公有成员和保护成员都变成派生类的保护成员
    + 基类的私有成员, 派生类无法访问.
- 需要注意的地方
    + 基类中的私有变量, 在派生类中都无法访问. **这里就体现了保护变量的用处, 既防止在外面被调用, 又可以让派生类访问到**
    + 私有成员无法在派生类中访问, 但是可以在派生类中调用基类中可以访问该私有成员的方法.
    + 派生类的对象都是基类的. 也就是说, 派生类的对象可以赋给基类的变量.P88

# 四. 多继承的定义格式
```c++
class <派生类名>: <继承方式><基类名1>,...,<继承方式><基类名n>
{
    <定义派生类自己的成员>
};
```
例如
```c++
class Derivedclass: public Baseclass1, public Baseclass2
{
    ...
};
```

# 五. 二义性
- ## 1. 调用不同基类的相同成员时可能会出现二义性(就是所继承的时候, 多个基类之间有名字相同的成员)
```c++
#include <iostream>

using namespace std;

class Baseclass1
{
public:
    void seta(int x){
        a = x;
    }
    void show(){
        cout<<"a = "<<a<<endl;
    }
private:
    int a;
};

class Baseclass2
{
public:
    void setb(int x){
        b = x;
    }
    void show(){
        cout<<"b = "<<b<<endl;
    }
private:
    int b;
};

class Derivedclass: public Baseclass1, public Baseclass2
{
public:
    void show(){
        Baseclass1::show();
        Baseclass2::show();
    }
};

int main()
{
    //错误的写法
    Derivedclass obj;
    obj.seta(2);
    obj.show();//error
    obj.setb(4);
    obj.show();//不知道是哪个 show ,存在二义性, 不能编译

    //修改方法一, 使用作用域运算符
    // Derivedclass obj;
    // obj.seta(2);
    // obj.Baseclass1::show();//error
    // obj.setb(4);
    // obj.Baseclass2::show();

    //修改方法二, 重写派生类中的 show 方法
    // Derivedclass obj;
    // obj.seta(2);
    // obj.setb(4);
    // obj.show();


    return 0;
}
```
- ## 2. 访问共同基类的成员时可能出现二义性
```c++
#include <iostream>

using namespace std;

/*
调用不同基类的相同成员可能出现二义性

爷爷类: Base
父类: Baseclass1, Baseclass2
子类: Derivedclass
爷爷类中有一个变量 val, Derivedclass 继承了两个父类, 而两个父类又都继承自爷爷类
导致两个父类都有 val, 子类中也有, 所以存在二义性.
解决方法是:
    Baseclass1::val
    Baseclass2::val
错误的做法是:
    Base::val
*/

class Base
{
protected:
    int val;
};


class Baseclass1: public Base
{
public:
    void seta(int x){
        val = x;
    }
};


class Baseclass2: public Base
{
public:
    void setb(int x){
        val = x;
    }
};


class Derivedclass: public Baseclass1, public Baseclass2
{
public:
    void show(){
        //cout<<"val = "<<val;    //含义不清, 不能编译

        //可以这样
        cout<<"val = "<<Baseclass1::val<<endl;
        //或者
        cout<<"val = "<<Baseclass2::val<<endl;

        //这样不行
        //cout<<"val = "<<Base::val<<endl;
    }
};


int main()
{
    Derivedclass obj;
    obj.seta(2);
    //obj.show();
    obj.setb(4);
    obj.show();

    return 0;
}
```
- ## 3. 二义性检查先于访问权限检查
```c++
#include <iostream>

using namespace std;


/*
访问权限和二义性的关系

1 里面有个 show 函数, 2 里面也有个 show 函数
但是 1 里面的 show 是公有的, 2 里面的是私有的
Derivedclass 继承了 1 和 2 函数, 当直接 obj.show() 的时候还是存在二义性
所以就说明了, 通过修改成员的访问权限并没有什么卵用.
因为二义性检查是在权限检查之前的
*/

class Baseclass1
{
public:
    void seta(int x){
        a = x;
    }

    void show(){
        cout<<"a = "<<a<<endl;
    }
private:
    int a;
};


class Baseclass2
{
public:
    void setb(int x){
        b = x;
    }
private:
    int b;
    void show(){
        cout<<"b = "<<b<<endl;
    }
};


class Derivedclass: public Baseclass1, public Baseclass2
{
};


int main()
{
    Derivedclass obj;
    obj.seta(2);
    obj.setb(4);
    obj.show();
    // obj.Baseclass1::show();

    return 0;
}
```

# 六. 虚基类
- ## 1. 目的
    + 解决二义性问题
    + 使得公共基类在派生类对象中只产生一个基类子对象
- ## 2. 格式
```c++
virtual <继承方式> <基类名>
```
    + 一个派生类可以公有或私有地继承多个虚基类, virtual 和 public 的顺序无关紧要,
但一定要放在基类名之前, 并且 virtual 只对紧随其后的基类名起作用.
- ## 3. 例子
```c++
#include <iostream>

using namespace std;


/*
虚基类解决二义性问题
爷爷类: A
父类: B1, B2
子类: C
B1, B2 继承 A, C 继承了 B1, B2, A 中有一个保护变量 val
如果不使用虚基类肯定出现二义性 
虚基类的使用方法:
    在定义派生类的时候, 加关键字 virtual
这样一来, A 中就只有一个 val 了
说来复杂, 在书本 98 页有张图片.
*/


class A
{
public:
    void setval(int x){
        val = x;
    }
protected:
    int val;
};


class B1: virtual public A
{
};


class B2: virtual public A
{
};


class C: public B1, public B2
{
public:
    void show(){
        cout<<"val = "<<val<<endl;
    }
};

int main()
{
    C c;
    c.setval(5);
    c.show();

    return 0;
}
```

# 七. 继承机制下的构造函数
- ## 1. 派生类中构造函数的格式
```c++
<派生类名>::<派生类名>(总参数表): <基类名1>(参数表1),...,<基类名n>(参数表n)
{
    <派生类中数据成员初始化>
}
```
- ## 2. 单继承并包括子对象时, 构造函数的调用顺序(基类 > 子对象 > 派生类)
```c++
#include <iostream>

using namespace std;

/*
有 Base1, Base2, Base3 三个类, 它们都有构造函数
Derivedclass 继承了 Base1, 还包含了 Base2, Base3 两个子对象.
Derivedclass 的构造函数中, 调用了另外三个类的构造函数
顺序是: 基类, 子对象, 派生类
注意:
    子对象构造函数的调用是根据在派生类中的声明顺序, 而不是派生类构造函数中出现的顺序
    派生类中的构造函数声明的时候可以不写出要调用的构造函数
*/

class Base1
{
public:
    Base1(int i){
        a = i;
        cout<<"Base1 的构造函数被调用, 此时 a 等于: "<<a<<endl;
    }
private:
    int a;
};


class Base2
{
public:
    Base2(int i)
    {
        b = i;
        cout<<"Base2 的构造函数被调用, 此时 b 等于: "<<b<<endl;
    }
private:
    int b;
};


class Base3
{
public:
    Base3(int i)
    {
        c = i;
        cout<<"Base3 的构造函数被调用, 此时 c 等于: "<<c<<endl;
    }
private:
    int c;
};


class Derivedclass: public Base1
{
public:
    Derivedclass(int i, int j, int k, int m);
private:
    int d;
    Base2 f;
    Base3 g;
};


Derivedclass::Derivedclass(int i, int j, int k, int m):Base1(i),g(j),f(k)
{
    d = m;
    cout<<"Derivedclass 的构造函数被调用, 此时 d 等于: "<<d<<endl;
}

int main()
{
    Derivedclass x(5, 6, 7, 8);

    return 0;
}
```
- ## 3. 多继承机制下构造函数的调用顺序(仍然是: 基类 > 子对象 > 派生类)
```c++
#include <iostream>

using namespace std;

/*
多继承方式下构造函数的调用顺序
基类>子对象>派生类
同一层次下的构造函数调用顺序取决于定义派生类时的继承顺序
*/


class Base1
{
public:
    Base1(int i){
        a = i;
        cout<<"Base1 的构造函数被调用, a = "<<a<<endl;
    }
private:
    int a;
};


class Base2
{
public:
    Base2(int i){
        b = i;
        cout<<"Base2 的构造函数被调用, b = "<<b<<endl;
    }
private:
    int b;
};


class Derivedclass: public Base1, public Base2
{
public:
    Derivedclass(int i, int j, int k);
private:
    int d;
};


Derivedclass::Derivedclass(int i, int j, int k):Base2(i),Base1(j)
{
    d = k;
    cout<<"Derivedclass 的构造函数被调用, d = "<<d<<endl;
}

int main()
{
    Derivedclass x(5, 6, 7);

    return 0;
}
```
- ## 4. 有虚基类时, 多继承方式下构造函数的调用顺序(虚基类 > 基类 > 派生类)(但虚基类的构造函数只调用一次)
```c++
#include <iostream>

using namespace std;

/*
有虚基类时, 多继承方式下构造函数的调用顺序

先要搞清楚几个类之间的关系, p105 有图
方法:
    调用一个类的构造函数, 只要这个类有基类, 就一定要先调用基类的构造函数.
    调用顺序按照继承顺序, 但是虚基类最优先.
    如果虚基类已经被调用一次, 再次出现就跳过.
*/

class Base1
{
public:
    Base1(){
        cout<<"Base1 的构造函数被调用."<<endl;
    }
};


class Base2
{
public:
    Base2(){
        cout<<"Base2 的构造函数被调用."<<endl;
    }
};


class Derived1:public Base2, virtual public Base1
{
public:
    Derived1(){
        cout<<"Derived1 的构造函数被调用."<<endl;
    }
};


class Derived2:public Base2, virtual public Base1
{
public:
    Derived2(){
        cout<<"Derived2 的构造函数被调用."<<endl;
    }
};


class Derived3: public Derived1, virtual public Derived2
{
public:
    Derived3(){
        cout<<"Derived3 的构造函数被调用."<<endl;
    }
};

int main()
{
    Derived3 obj;

    return 0;
}
```
# 八. 派生类构造函数的规则
- ## 1. 基类无, 派生类有
```c++
#include <iostream>

using namespace std;


/*
基类没有构造函数时, 派生类构造函数的规则
(完全没规则吧???)
*/

class Baseclass
{
private:
    int a;
};


class Derivedclass: public Baseclass
{
public:
    Derivedclass();
    Derivedclass(int i);
private:
    int b;
};


Derivedclass::Derivedclass()
{
    cout<<"Derivedclass 的默认构造函数被调用."<<endl;
}

Derivedclass::Derivedclass(int i)
{
    b = i;
    cout<<"Derivedclass 的构造函数被调用, b = "<<b<<endl;
}


int main()
{
    Derivedclass x1(5);
    Derivedclass x2;

    return 0;
}
```
- ## 2. 基类有, 派生类无(调用派生类的默认构造函数, 然后执行基类的构造函数, 如果基类没有默认构造函数就会出错)
```c++
#include <iostream>

using namespace std;

/*
基类有构造函数, 派生类没有自定义构造函数时的规则
派生类没有构造函数, 会调用默认的构造函数, 然后执行基类的默认构造函数
(为防止子类没有构造函数, 父类一定要有默认构造函数???)
*/

class Baseclass
{
public:
    Baseclass()
    {
        cout<<"Baseclass 默认构造函数被调用."<<endl;
    }
    Baseclass(int i)
    {
        a = i;
        cout<<"Baseclass 的构造函数被调用, a = "<<a<<endl;
    }
private:
    int a;
};


class Derivedclass: public Baseclass
{
private:
    int b;
};


int main()
{
    Derivedclass x;

    return 0;
}
```
- ## 3. 基类有, 派生类也有<!-- (如果派生类没有显式调用基类的构造函数, 默认构造函数被调用, 但如果刚好基类中没有默认构造函数, 就会报错)(如果显式调用就正常执行) -->
```c++
#include <iostream>

using namespace std;

/*
基类有构造函数, 派生类构造函数的规则
如果在派生类的构造函数没有调用基类的构造函数, 那么就调用基类的默认构造函数
如果在派生类的构造函数中调用了基类的构造函数, 那么就调用基类中对应的构造函数.
*/

class Baseclass
{
public:
    Baseclass()
    {
        cout<<"Baseclass 的默认构造函数被调用."<<endl;
    }
    Baseclass(int i)
    {
        a = i;
        cout<<"Baseclass 的构造函数被调用, a = "<<a<<endl;
    }
private:
    int a;
};


class Derivedclass: public Baseclass
{
public:
    Derivedclass(int i);
    Derivedclass(int i, int j);
private:
    int b;
};

Derivedclass::Derivedclass(int i)
{
    b = i;
    cout<<"Derivedclass 的构造函数 1 被调用, b = "<<b<<endl;
}

Derivedclass::Derivedclass(int i, int j): Baseclass(i)
{
    b = j;
    cout<<"Derivedclass 的构造函数 2 被调用, b = "<<b<<endl;
}

int main()
{
    Derivedclass x1(5, 6);
    Derivedclass x2(7);

    return 0;
}
```
- ## 4. 基类无默认构造函数时, 派生类构造函数的规则
```c++
#include <iostream>

using namespace std;

/*
基类无默认构造函数时, 派生类构造函数的规则
以下情况会出错:
    基类无默认函数, 派生类中有构造函数没有调用基类的其他构造函数
解决方法:
    在基类中加入默认构造函数
    在派生类的所有构造函数中调用基类的构造函
注意:
    即使没有调用派生类中那个 没有调用基类的其他构造函数的构造函数, 还是会出错.
*/

class Baseclass
{
public:
    Baseclass(){};
    Baseclass(int i)
    {
        a = i;
        cout<<"Baseclass 的构造函数被调用. a = "<<a<<endl;
    }
private:
    int a;
};


class Derviedclass: public Baseclass
{
public:
    Derviedclass(int i);
    Derviedclass(int i, int j);
private:
    int b;
};

Derviedclass::Derviedclass(int i)
{
    b = i;
    cout<<"Derviedclass 的构造函数 1 被调用, b = "<<b<<endl;
}

Derviedclass::Derviedclass(int i, int j):Baseclass(i)
{
    b = j;
    cout<<"Derviedclass 的构造函数 2 被调用, b = "<<b<<endl;
}

int main()
{
    Derviedclass x1(5, 6);
    //Derviedclass x2(7);

    return 0;
}
```

# 九. 继承机制下的析构函数
**刚好跟构造函数的调用顺序相反**
```c++
#include <iostream>

using namespace std;


/*
派生类析构函数的调用顺序
刚好与构造函数的调用顺序相反, 此程序有 warning
*/

class Base1
{
public:
    Base1(int i)
    {
        a = i;
        cout<<"Base1 的构造函数被调用, a = "<<a<<endl;
    }
    ~Base1()
    {
        cout<<"Base1 的析构函数被调用."<<endl;
    }
private:
    int a;
};


class Base2
{
public:
    Base2(int i)
    {
        b = i;
        cout<<"Base2 的构造函数被调用, b = "<<b<<endl;
    }
    ~Base2()
    {
        cout<<"Base2 的析构函数被调用."<<endl;
    }
private:
    int b;
};


class Base3
{
public:
    Base3(int i)
    {
        c = i;
        cout<<"Base3 的构造函数被调用, c = "<<c<<endl;
    }
    ~Base3()
    {
        cout<<"Base3 的析构函数被调用."<<endl;
    }
private:
    int c;
};


class Derivedclass: public Base1
{
public:
    Derivedclass(int i, int j, int k, int m);
    ~Derivedclass();
private:
    int d;
    Base2 f;
    Base3 g;
};

Derivedclass::Derivedclass(int i, int j, int k, int m):Base1(i),g(j),f(k)
{
    d = m;
    cout<<"Derivedclass 的构造函数被调用, d = "<<d<<endl;
}

Derivedclass::~Derivedclass()
{
    cout<<"Derivedclass 的析构函数被调用."<<endl;
}

int main()
{
    Derivedclass x(5, 6, 7, 8);

    return 0;
}
```

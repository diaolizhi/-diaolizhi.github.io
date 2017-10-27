---
title: C++ 运算符重载(一)
date: 2017-10-24
categories: C++
---

简单了解运算符重载.
<!--more-->

# 一. 为什么会有 「运算符重载」?
简单的运算符包括加减乘除, 对于系统自带的类型, 比如
*int* *float*
我们可以直接相加减, 比如 
```c++
int a=1, b=2, c; 
c=a+b;
```

如果希望我们自定义的类型也可以用 "+" 运算, 就需要到 「运算符重载」 .

(实际上很多运算符都可以重载.)

# 二. 什么是 「运算符重载」?
运算符的重载就是在类中**定义一个函数**, 对某个运算符进行重载.
比如:

```c++
#include <iostream>

using namespace std;

//定义绳子类
class rope
{
    public:
        //构造函数
        rope(int i=1)
        {
            len = i;
        }
        //重载 + 运算符
        rope operator + (rope& r)
        {
            len = len + r.len;
            return *this;
        }
        //输出绳子的长度
        void show()
        {
            cout<<"len: "<<len<<endl;
        }
    private:
        int len;
};

int main(int argc, char const *argv[])
{
    rope a(2), b(3);
    b = a + b;    //两根绳子相加
    b.show();
    
    return 0;
}
```

例子里自定义了一个绳子类, 属性有长度. 
```c++
rope operator + (rope& r)
{
    len = len + r.len;
    return *this;
}
```

其中实现了运算符的重载, 所以可以在主函数里对两个绳子对象进行 + 运算.

# 三. 怎么实现 「运算符重载」?
运算符的重载是通过**函数**来实现的, 一种是**成员函数**, 另一种是**友元函数**.
## 1. 用成员函数重载运算符
- 原型
    ```c++
    <返回值类型>operator<运算符>(<形式参数表>);
    ```
- 特点
    + 因为成员函数有一个 this 指针, 所以它需要的参数总是比它的操作时**少一个**.
    + 在运算符重载之后
        ```c++
        a + b
        ```
        会被编译器解释为 
        ```c++
        a.operator +(b)
        ```
- 例子
    + 不想写.


## 2.用友元函数重载运算符
- 原型
    ```c++
    friend<返回值类型>operator<运算符>(<形式参数表>)
    ```
- 特点
    + 由于友元函数不是类的成员, 没有 this 指针, 因此需要的参数和操作数一样多.
    + 在运算符重载之后
        ```c++
        a + b
        ```
        会被编译器解释为 
        ```c++
        operator +(a, b)
        ```
        这个例子还不能看出友元函数的功能, 再来一个
        ```c++
        23.5 + b
        ```
        会被编译器解释为 
        ```c++
        operator +(23.5, b)
        ```
        如果像成员函数那样, 就会被解释为 
        ```c++
        23.5.operator(b)
        ```
        而 float 类型不能进行"."操作. 所以就错了.
- 例子
    + 不想写.

## 3. 两种运算符重载形式的比较
- 友元函数重载运算符的特点已经体现出**有时候重载为友元函数更加灵活**.
- **不能**将**"="、 "+="、 "-="**等赋值运算符重载为友元函数．
- **不能**将**"()"、 "[]"、 "->"**重载为友元函数.
- 在重载增量或减量运算符时, 若使用友元函数, 则需要应用**引用参数**.


# 四. 不同运算符的重载
## 1. 加减乘除运算符的重载
加减乘除相对其他简单, 以**成员函数**为例, 在类中的原型为:
```c++
<返回值类型>operator + (<形式参数表>);
<返回值类型>operator - (<形式参数表>);
<返回值类型>operator * (<形式参数表>);
<返回值类型>operator / (<形式参数表>);
```
然后根据需要进行操作即可.(参考**第一个程序**即可, 在此不再举例. 也可用友元函数重载加减乘除运算符, 也懒得写)
## 2. 单目运算符的重载
单目运算符只有一个操作数, 如 !a, -b, &c, \*p, \-\-i, ++j.
这里以**增量运算符"++"及减量运算符"\-\-"**为例, 介绍单目运算符的重载.
### (1) 用**成员函数**重载 ++ 和 \-\-
- 原型
    ```c++
    <返回值类型>::operator ++();       //前缀 ++ 运算符
    <返回值类型>::operator ++(int);    //后缀 ++ 运算符
    <返回值类型>::operator --();       //前缀 -- 运算符
    <返回值类型>::operator --(int);    //后缀 -- 运算符
    ```
- 特点
    + int 参数表明调用**该函数时运算符"++"应放在操作数**的后面, 而且这个参数并没有使用到, 所以没必要给出参数名字.
    + 因为成员函数有一个 this 指针, 所以不必要再给出其他参数.

- 例子
    ```c++
#include <iostream>

using namespace std;

/*
2017年10月16日 18:46:46
成员函数重载运算符 ++ 和 --
*/

class Counter
{
public:
    //构造函数
    Counter () {};
    Counter (int i) {value = i;}
    //重载 ++ --
    Counter operator ++ ();
    Counter operator ++ (int);
    Counter operator -- ();
    Counter operator -- (int);
    //打印数据成员
    void display() {cout<<value<<endl;}
private:
    unsigned value;
};

Counter Counter::operator ++ ()
{
    value ++;    //++
    return * this;    //把当前对象返回回去
}

Counter Counter::operator ++ (int)
{
    Counter temp;    //临时对象
    temp.value = value++;    //临时对象的 value 等于实参的 value, 然后实参的 value 加一
    return temp;    //把临时对象返回回去
    /*
        后缀 ++ 和前缀 ++ 之所以不同, 是因为前缀本来就是先 ++, 而后缀要运算之后才 ++.
        如果在后缀 ++ 里面, 直接返回 *this, 那么在主函数里面接收到的是错的. 
    */
}

Counter Counter::operator -- ()
{
    value --;
    return * this;
}

Counter Counter::operator -- (int)
{
    Counter temp;
    temp.value = value --;
    return temp;
}

int main()
{
    Counter n(10), c;

    c = ++ n;
    cout<<"前缀 ++ 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = n++;
    cout<<"后缀 ++ 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = -- n;
    cout<<"前缀 -- 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = n--;
    cout<<"后缀 -- 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();


    return 0;
}
    ```

### (2) 用**友元函数**重载 ++ 和 \-\-
- 原型
    ```c++
    <类型>operator ++ (<类名>&);
    <类型>operator -- (<类名>&);
    <类型>operator ++ (<类名>&, int);
    <类型>operator -- (<类名>&, int);
    ```
- 特点
    + 因为友元函数**没有** this 指针, 所以参数里面要把实参传过来.
    + 本来 ++ 和 -- 是会对原来的数产生影响的, 所以重载的时候也要保证这一点, 于是参数中传入的就要是**引用**, 这样就可以改变主函数里面的对象.
- 例子
    ```c++
#include <iostream>

using namespace std;

/*
2017年10月16日 18:56:20
友元函数重载运算符 ++ 和 --
*/

class Counter
{
public:
    Counter () {value = 0;}
    Counter (int i) {value = i;}
    friend Counter operator ++ (Counter &);
    friend Counter operator ++ (Counter &, int);
    friend Counter operator -- (Counter &);
    friend Counter operator -- (Counter &, int);
    void display() {cout<<value<<endl;}
private:
    unsigned value;
};

Counter operator ++ (Counter & p)
{
    p.value++;
    return p;
}

Counter operator ++ (Counter & p, int)
{
    Counter temp;
    temp.value = p.value++;
    return temp;
}

Counter operator -- (Counter & p)
{
    p.value--;
    return p;
}

Counter operator -- (Counter & p, int)
{
    Counter temp;
    temp.value = p.value--;
    return temp;
}

int main()
{
    Counter n(10), c;

    c = ++ n;
    cout<<"前缀 ++ 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = n++;
    cout<<"后缀 ++ 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = -- n;
    cout<<"前缀 -- 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    c = n--;
    cout<<"后缀 -- 运算符计算结果:"<<endl;
    cout<<"n = ", n.display();
    cout<<"c = ", c.display();

    return 0;
}
    ```

## 3.赋值运算符的重载
### (1) 浅拷贝和深拷贝
- 浅拷贝: 当把一个对象赋值给另一个对象的时候, 相当于值的赋值. 如果对象里有一个指针, 也是把**指针的值复制**过去. 问题是 如果在析构函数里面释放指针所指向的内存, 那么程序结束的时候, 对象 A 释放了指针所指向的空间, 但是 B 对象的指针是和 A 中指针一样的, 所以**再次释放**, 于是就出错了.(C-Free 会忽略此错误)
- 深拷贝: 我们可以重载赋值运算符(也就是 =), 使得在赋值的时候 B 的指针**指向一块新的内存**, 具体看例子.
- 浅拷贝的例子
```c++
#include <iostream>

using namespace std;
/*
2017年10月16日 19:28:39
错误的浅拷贝
在 C-Free 会忽略这个错误, 在 VC 就会报错.
*/

class Namelist
{
    char * name;    //定义了一个指针
public:
    Namelist(int size)
    {
        name = new char[size];
    }
    ~Namelist()
    {
        delete [] name;    //在析构函数里面释放了这个指针
    }
};

int main()
{
    Namelist n1(10), n2(20);

    n2 = n1;    //没有重载赋值运算符, 就会把 n1 的指针的值赋给 n2 的指针

    return 0;
}
```
### (2) 重载赋值运算符的格式
```c++
Classname& Classname::operator = (Classname& obj)
{
    if(this != &obj)
    {
        delete ptr;    //释放原来分配的空间
        //根据 obj 分配新的空间
        //如果分配的空间不为 NULL, 把 obj　的值复制过来
    }
    return *this;
}
```
- 注意
    + 形式参数要是**引用**, 否则会调用拷贝构造函数, 又出现浅拷贝.(个人理解)
    + 重载赋值运算符函数的返回值
    >*重载赋值运算符时, 通常是返回调用该运算对象的引用, 不过不能使用引用返回一个局部变量, 但 this 可以解决这个问题.*

        + 为什么返回引用 ? 如果不是返回引用就不能**连续赋值**.
        + 为什么书上说**不能使用引用返回一个局部变量**, 而重载单目运算符的时候返回的是**局部变量** ? 
            * 返回局部变量相当于把局部变量**复制**给主调函数里面的变量, 并不是一定会出问题.(浅拷贝没问题就没问题吧)
            * 而返回引用的前提下, 主调函数指向临时变量, 而这个临时变量是**临时**的, 函数调用之后就**释放**了, 所以就不行.
    + 简单来说, 就是要记住**形式参数是引用, 返回类型也是引用, return 的是 \*this.**
- 深拷贝的例子
```c++
#include <iostream>
#include <cstring>

using namespace std;

/*
2017年10月25日 22:18:03
深拷贝的例子
*/

class Namelist
{
    public:
        //构造函数
        Namelist(char *p)
        {
            str = new char[strlen(p)];
            if(str != NULL)
            {
                strcpy(str, p);
            }
        }
        //重载赋值运算符
        Namelist& operator = (Namelist& n)
        {
            if(this != &n)    //如果是 A = A 这样的赋值语句没意思了
            {
                delete str;
                str = new char[strlen(n.str)];
                if(str != NULL)
                {
                    strcpy(str, n.str);
                }
            }
            return *this;
        }
        void show()
        {
            cout<<str<<endl;
        }
    private:
        char * str;
};


int main(int argc, char const *argv[])
{
    char aa[] = "aaa", bb[] = "bbb", cc[] = "ccc";
    Namelist a(aa), b(bb), c(cc);

    cout<<"赋值前: "<<endl;
    a.show();
    b.show();
    c.show();

    a = b = c;
    cout<<"赋值后: "<<endl;
    a.show();
    b.show();
    c.show();
    
    return 0;
}
```

### (3) 和拷贝构造函数的区别
拷贝构造函数和赋值运算符重载函数都是用来拷贝一个对象给另一个同类型的对象, 不过它们还是有些区别的.
- 拷贝构造函数是用一个**已存在的**对象去初始化一个**新的**对象, 在以下三种情况下被调用:
    + 声明一个对象的同时, 给它赋值另一个已存在的对象.
    + 对象作为函数参数, 实参和形参结合时.
    + 函数返回值是类的对象, 函数调用结束后返回到主调函数
- 赋值运算符重载函数是把一个**已存在的**对象赋值给另一个**已存在的**同类型对象时调用.

## 4.特殊运算符的重载
### (1)[ ] 的重载
- 格式
```c++
<类型>operator [] (int);
```
- 特点
    + [] 只能重载为**成员函数**, 不可重载为友元函数
    + 类中重载了下标运算符"[]", 则可将该类的对象当作一个**数组**
    + **必须且只能带一个形参**, 该形参规定为**整数**
    + 重载 [] 可以实现**安全数组下标**
    + 如果返回的是**引用**, 还可以作为**左值**
- 例子
```c++
#include <iostream>
#include <process.h>
#include <stdlib.h>

/*
2017年10月16日 20:49:44
重载下标运算符[]
*/

using namespace std;

const int LIMIT = 100;    //定义数组的最大长度

class Intarray
{
private:
    int size;   //数组的实际长度
    int * array;    //数组首指针
public:
    Intarray(int = 1);    //构造函数, 默认数组长度为 1
    int& operator [] (int i);    //重载下标运算符
    ~Intarray();    //析构函数
};

Intarray::Intarray(int i)
{
    //数组越界检查
    if(i<0 || i>LIMIT)
    {
        //如果长度小于 0 或者 超过了最大长度就退出程序
        cout<<"下标越界!"<<endl;
        exit(1);
    }
    size = i;
    array = new int[size];    //动态分配空间
    for(int j=0; j<i; j++)
    {
        array[j] = j*10;
    }
}

int& Intarray::operator [] (int i)
{
    if(i<0 || i>size-1)
    {
        //如果下标不合法就退出程序
        // cout<<"log error"<<endl;
        cout<<"下标越界!"<<endl;
        exit(1);
    }
    // cout<<"log"<<endl;
    return array[i];
};

Intarray::~Intarray()
{
    delete[] array;
    size = 0;
}


int main()
{
    int k, num;
    cout<<"请输入数组大小:";
    cin>>k;

    Intarray array(k);
    
    cout<<"请输入要打印的个数:";
    cin>>num;
    for(int j=0; j<num; j++)
    {
        int temp = array[j];
        cout<<"元素 array["<<j<<"] : "<<temp<<endl;
    }

    return 0;
}
```
### (2)( ) 的重载
- 格式
```c++
<类型>operator () (<参数表>);
```
- 特点
    + 只能重载为**成员函数**
    + 其中的类型可以使**任意类型**, 参数表可以是**任意多个**, 包括 0 个
    + >当重载"()"调用运算符时, 并不是创建一种调用函数的新方法, 而是创建可传递任意多个参数的**运算符函数**
- 例子 1
```c++
#include <iostream>

using namespace std;

/*
2017年10月17日 15:27:51
重载 ()
*/

class Func
{
private:
    double x, y, z;    //定义成员变量
public:
    Func(double x, double y, double z)
    {
        //构造函数
        this->x = x;
        this->y = y;
        this->z = z;
    }
    double getx(){return x;}
    double gety(){return y;}
    double getz(){return z;}
    double operator () (double x, double y, double z);    //重载 ()
};

double Func::operator () (double x, double y, double z)
{
    //重载 () 可以传入任意多个参数, 想干嘛都行
    return x + y + z;
}

int main()
{
    Func fc(1, 2, 3);

    cout<<"("<<fc.getx()<<","<<fc.gety()<<","<<fc.getz()<<")"<<endl;

    cout<<fc(1, 2, 3)<<endl;

    return 0;
}
```

- 例子 2
```c++
#include <iostream>
#include <process.h>
#include <stdlib.h>

using namespace std;

/*
2017年10月17日 15:28:13
重载 () 访问二维数组
*/

const int LIMIT = 100;    //数组最大长度

class Intarray
{
private:
    int size1;    //行
    int size2;    //列
    int * array;    //数组指针
public:
    Intarray(int = 1, int = 1);    //默认为 1 行 1 列
    int& operator () (int i, int j);
    ~Intarray();
};

Intarray::Intarray(int i, int j)
{
    if((i<0 || i>LIMIT) || (j<0 || j>LIMIT))
    {
        //如果数组长度不合法就退出程序
        cout<<"下标越界!"<<endl;
        exit(1);
    }
    size1 = i;
    size2 = j;
    array = new int[i*j];
    //给数组赋值
    for(int x=0; x<i; x++)
    {
        for(int y=0; y<j; y++)
        {
            array[x*size1+y] = 2*x+y;
        }
    }
}

int& Intarray::operator () (int i, int j)
{
    if((i<0 || i>=LIMIT) || (j<0 || j>=LIMIT))
    {
        //检查下标
        cout<<"下标越界!"<<endl;
        exit(1);
    }

    return array[i*size1+j];
}

Intarray::~Intarray()
{
    delete[] array;
    size1 = 0;
    size2 = 0;
}

int main(int argc, char const *argv[])
{
    int r, c, m, n, i, j;
    cout<<"请输入二维数组个数:"<<endl;
    cin>>r>>c;
    Intarray array(r, c);

    cout<<"请输入要输出的个数:"<<endl;
    cin>>m>>n;

    for(i=0; i<m; i++)
    {
        for(j=0; j<n; j++)
        {
            int temp = array(i, j);
            cout<<"元素 "<<"("<<i<<","<<j<<") "<<temp<<endl;
        }
    }


    return 0;
}
```

## 5. 类类型转换运算符重载
### (1)基本类型 to 类类型
- 注意
    + 利用**构造函数**可以完成基本类型到类类型的转换, 前提是: 类中一定要具有最多只有一个非默认参数的构造函数.(比如复数类重载了 + 运算符, 当执行 c+11 的时候, 由于 11 不属于复数, 所以会通过复数类的构造函数将 11 转为复数类型, 书上叫做建立**临时对象**. 此时, **11 只能作为一个参数**, 而类中必须要有合适的构造函数能被调用才行.)
- 例子
    + 没有

### (2)类类型 to 基本类型
- 格式
```c++
operator<返回类型名>()
{
    ...
    return <基本类型值>
}
```
- 特点
    + 类类型转换函数专门将**类类型转换为基本数据类型**的, 它只能重载为**成员函数**.
    + 和以前的重载运算符函数不同, 类类型转换运算符重载是没有返回值类型, 因为<返回类型名>代表的就是它的返回值类型.
- 例子 1
```c++
#include <iostream>

using namespace std;

/*
2017年10月17日 15:29:11
重载类类型转换运算符
*/

class Rtype
{
public:
    Rtype(int a, int b = 1);
    operator double();
private:
    int data1;
    int data2;
};

Rtype::Rtype(int a, int b)
{
    data1 = a;
    data2 = b;
}

Rtype::operator double()
{
    //把分数转换为小数
    return double(data1)/double(data2);
}

int main(int argc, char const *argv[])
{
    Rtype r1(2, 4), r2(3, 8);
    cout<<"r1 = "<<double(r1)<<", r2 = "<<double(r2)<<endl;
    
    return 0;
}
```

例子 2
```c++
#include <iostream>

using namespace std;

class Rr
{
public:
    Rr(int a)
    {
        data = a;
    }
    operator double()
    {
        return double(data);
    }
    operator int()
    {
        return int(data);
    }
private:
    int data;
};

int main(int argc, char const *argv[])
{
    Rr r1(2), r2(3);
    int x = int(r1) + int(r2);
    float y = double(r1) / double(r2);
    cout<<"x = "<<x<<", y = "<<y<<endl;
    
    return 0;
}
```

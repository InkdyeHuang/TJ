<!-- GFM-TOC -->
* [重写、重载、隐藏](#重写-重载-隐藏)
<!-- GFM-TOC -->

###重写、重载、隐藏
- 重载(overload)：同一访问区间内声明具有不同参数列表(参数的类型、个数和顺序)的同名函数,根据参数列表确定调用。重载不关心函数的返回类型，即不能有同名同参数列表但是参数返回类型不同的函数。
```c++
class A{
public:
  void test(int i);
  void test(double i);//overload
  void test(int i, double j);//overload
  void test(double i, int j);//overload
  int test(int i);         //错误，非重载。注意重载不关心函数返回类型。
};
```

- 重写(覆盖)：是指派生类中存在重新定义的函数（该函数和基类函数同名同参同返回值，且是虚函数，用virtual修饰）。
```c++
#include<iostream>

using namespace std;

class Base
{
public:
    virtual void fun(int i){ cout << "Base::fun(int) : " << i << endl;}
};

class Derived : public Base
{
public:
    virtual void fun(int i){ cout << "Derived::fun(int) : " << i << endl;}
};
int main()
{
    Base b;
    Base * pb = new Derived();
    pb->fun(3);//Derived::fun(int)

    system("pause");
    return 0;
}
```
- 隐藏(重定义)：是指派生类的函数屏蔽了与其同名的基类函数，注意只要同名函数，不管参数列表是否相同，基类函数都会被隐藏。除了重写的行为。

```c++
#include "stdafx.h"
#include "iostream"

using namespace std;

class Base
{
public:
    void fun(double ,int ){ cout << "Base::fun(double ,int )" << endl; }
};

class Derive : public Base
{
public:
    void fun(int ){ cout << "Derive::fun(int )" << endl; }
};

int main()
{
    Derive pd;
    pd.fun(1);//Derive::fun(int )
    pb.fun(0.01, 1);//error C2660: “Derive::fun”: 函数不接受 2 个参数

    Base *fd = &pd;
    fd->fun(1.0,1);//Base::fun(double ,int);
    fd->fun(1);//error 
    system("pause");
    return 0;
}
```
- 重载和重写的区别：
（1）范围区别：重写和被重写的函数在不同的类中，重载和被重载的函数在同一类中。
（2）参数区别：重写与被重写的函数参数列表一定相同，重载和被重载的函数参数列表一定不同。
（3）virtual的区别：重写的基类必须要有virtual修饰，重载函数和被重载函数可以被virtual修饰，也可以没有。

- 隐藏和重写，重载的区别：
（1）与重载范围不同：隐藏函数和被隐藏函数在不同类中。
（2）参数的区别：隐藏函数和被隐藏函数参数列表可以相同，也可以不同，但函数名一定同；当参数不同时，无论基类中的函数是否被virtual修饰，基类函数都是被隐藏，而不是被重写。
<!-- GFM-TOC -->
* [修饰符](#修饰符)
    * [const](#const)
    * [extern](#extern)
* [指针](#指针)
* [面向对象](#面向对象)
* [重写、重载、隐藏](#重写-重载-隐藏)
* [构造函数](#构造函数)
* [STL](#stl)

<!-- GFM-TOC -->
## 修饰符
### const
const只修饰离他最近的类型符号 
在全局作用域中，const定义的变量是本文件的局部变量，只能再本文件中访问。
```c++
//file1
extern const int bufsize;
//file2，用extren可以访问另一个文件的常量。
extern const int bufsize;

typedef string *  pstring;
const pstring cstr;
//等价于，typedef不是简单的字符扩展，而是代表一个类型。
string * const cstr;

const int **p; int const **p; // 修饰**p代表的整型内存放的是常量。
int * const * p;//修饰的是int *，一级指针*p是常量
int ** const p; //修饰二级指针p本身是常量
```

## 指针
###数组指针与指针数组
数组指针(行指针): int (*p)[n]; ()优先级高，说明p是一个指针，指向一个一维数组，以为数组的指针步长为n,p+1表示，p要跨过n个整型数据的长度。
```c++
int a[3][4];
int(*p)[4];
p = a;//将二位数组的首地址赋给了p；
p++;//p跨过行a[0][]指向a[1][]
```
指针数组: int* p[]; []优先级高，先成为一个数组，而int*则说明这是一个整型指针数组。
```c++
int a[3][4];
int *p[4];
p = a; //error, p是未可知的表示,p表示指向指针数组的常量指针
for(int i = 0; i < 3; i++)
    p[i] = a[i];
```
函数： int *p();()优先级最高，所以是个函数，返回值为int *，返回值为指针的函数。
函数指针 int (*p)();()优先级高从左到右，所以p是一个指针，指向类型为int ()的函数。
函数指针数组：int (*a[10]) (int); []优先级最高，所以a是一个数组，类型是个指针数组，指针指向函数为int (int);
'''c++
#include <iostream>
#include <stdio.h>
using namespace std;

int func1(int n)
{
    printf("func1: %d\n", n);
    return n;  
}
int func2(int n)
{
    printf("func2: %d\n", n);
    return n;  
}
int main()
{
    int (*a[10])(int) = {NULL};
    a[0] = func1;
    a[1] = func2;
    a[0](1);
    a[1](2);  
    return 0;    
}
'''

### 野指针和悬空指针
* 野指针——没有被初始化的指针。
```c++
int main()
{
    int *p;
    return p & 0x07;
}
```
* 悬空指针——指向的内存已经被释放了的一种指针
## 面向对象
### 重写、重载、隐藏
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

### 构造函数
#### 构造函数为什么不能是虚函数
1. 虚函数对于一个虚函数表，存储在对象的内存空间，如果构造函数时序的。对象没有实习化，无法调用虚函数表，无法实例化。
2. 实现上vbtl在构造函数调用后才建立，因而构造函数不可能成为虚函数  
3. 当一个构造函数被调用时，首先要是初始化VP R。而只能知道它是“当前”类的，而完全忽视这个对象后面是否还有继承者。当编译器为这个构造函数产生代码时，它是为这个类的构造函数产生代码- -既不是为基类，也不是为它的派生类（因为类不知道谁继承它）。
3. 构造函数不需要是虚函数，也不允许是虚函数，因为创建一个对象时总要明确指定对象的类型，尽管可以通过基类的指针或引用去访问它。但析构却不一定，我们往往通过基类的指针来销毁对象。这时候如果析构函数不是虚函数，就不能正确识别对象类型从而不能正确调用析构函数。——基类的析构函数一般都为虚函数。
#### 拷贝构造函数为什么传引用
值传递：
1.内置类型传递时，直接赋值拷贝给形参（形参时内部变量）。
2.类类型传递，会先调用该类的拷贝构造函数初始化新参（局部变量）。
引用传递：
无论对内置类型还是类类型，传递引用或指针最终都是传递的地址值！而地址总是指针类型(属于简单类型), 显然参数传递时，按简单类型的赋值拷贝，而不会有拷贝构造函数的调用。
* 拷贝构造函数传值的话会造成无限调用。
#### 默认拷贝拷贝构造函数可能存在的问题，及类成员存在指针的时候，要显式声明拷贝构造函数。
    如果不显式声明拷贝构造函数的时候，编译器也会生成一个默认的拷贝构造函数，而且在一般的情况下运行的也很好。但是在遇到类有指针数据成员时就出现问题了：因为默认的拷贝构造函数是按成员拷贝构造，这导致了两个不同的指针(如ptr1=ptr2)指向了相同的内存。当一个实例销毁时，调用析构函数free(ptr1)释放了这段内存，那么剩下的一个实例的指针ptr2就无效了，在被销毁的时候free(ptr2)就会出现错误了, 这相当于重复释放一块内存两次。这种情况必须显式声明并实现自己的拷贝构造函数，来为新的实例的指针分配新的内存。
## STL
### it++和++it
    ++it直接返回引用，it++会返回一个无用的临时对象，而且因为是函数重载，编译器无法优化，没遍历一遍都要删除一遍无用的临时遍历，所以++it更好。
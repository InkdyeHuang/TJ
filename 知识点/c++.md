<!-- GFM-TOC -->
* [修饰符](#修饰符)
    * [const](#const)
    * [extern](#extern)
* [指针](#指针)
* [面向对象](#面向对象)
* [重写、重载、隐藏](#重写-重载-隐藏)
* [构造函数](#构造函数)
* [虚函数](#虚函数)
* [STL](#stl)
* [c++11新特性](#c++11新特性)

<!-- GFM-TOC -->
## 修饰符
### const
const作用:定义const常量，保护被修饰的对象，防止修改，const修饰的对象存在常量存储区（符号表）只有一份拷贝，是编译期间的常量，提升了程序的效率和健壮性。
#### const 修饰变量
const只修饰离他最近的类型符号；在全局作用域中，const定义的变量是本文件的局部变量，只能再本文件中访问。
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
#### const修饰函数
常成员函数，const修饰函数，只能是成员函数，而且为重载提供可能,但是常量成员函数只有权读取外部数据内容，无权修改。

作用：当函数体较大且复杂时，如果希望系统帮助避免修改对象内容，那么可以将这个函数定义为常量型函数
```c++
class A
{
    const int nValue;//const 修饰类成员变量，只能在初始化表中赋值或者直接赋值。
    const int nValue = 1;
    A(int x):nValue(x){};
    void f(int i){...}
    void f(int i) const{...} //上一个函数的重载
    const int * fun2(){...}//调用时 const int * pValue = fun2()
    int * const fun3(){...} //调用时 int * const pValue = fun3()
};
```
### const与#define 对比

1. const修饰的都带有类型（有类型检测），#define 修饰的只是个常量没有类型检测.
2. #define 在编译预处理阶段起作用，是代码段的简单字符替换，const在编译、运行时起作用； const可以进行调试，#define不可以进行调试，因为预编译阶段就替换掉了。
3. const定义的常量都存在常量存储区(符号表)，只有一份拷贝，而#define定义的常量内存中有多份拷贝，预处理后占用代码段空间。
4. const 不能重复定义， #define可以通过#undef取消掉某个符号的定义，然后再重新定义。
```
#define N 2+4 //预处理后占用代码段区间
int a = N/2; //简单的字符串替换 a = 4
const float PI = 3.14；//本质上是一个float，占用数据段空间
```
### const_cast
强制类型转化:const_cast <type_id>  (expression) 将const转成非const
```c++
const int a = 1;
int * p = const_cast<int *>(&a);
*p = 2;
cout<<*p<<" "<<a<<endl;//*p=2 , a=1;
const int *aa = &a;
int* pp = const_cast<int*>(aa);
*pp = 2;
cout<<*aa<<endl;//2
```

### extern
作用声明外部变量或者函数，可以声明多次。

全局变量可以用extren声明，局部变量不能用extern修饰，且局部变量运行时才在栈里分配内存。

## 指针
### 数组指针与指针数组
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

函数指针： int (*p)();()优先级高从左到右，所以p是一个指针，指向类型为int ()的函数。

函数指针数组：int (*a[10]) (int); []优先级最高，所以a是一个数组，类型是个指针数组，指针指向函数为int (int);

```c++
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
```

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
#### 拷贝构造函数为什么传引用
值传递：
1.内置类型传递时，直接赋值拷贝给形参（形参时内部变量）。
2.类类型传递，会先调用该类的拷贝构造函数初始化新参（局部变量）。
引用传递：
无论对内置类型还是类类型，传递引用或指针最终都是传递的地址值！而地址总是指针类型(属于简单类型), 显然参数传递时，按简单类型的赋值拷贝，而不会有拷贝构造函数的调用。
* 拷贝构造函数传值的话会造成无限调用。
#### 默认拷贝拷贝构造函数可能存在的问题，及类成员存在指针的时候，要显式声明拷贝构造函数。
    如果不显式声明拷贝构造函数的时候，编译器也会生成一个默认的拷贝构造函数，而且在一般的情况下运行的也很好。但是在遇到类有指针数据成员时就出现问题了：因为默认的拷贝构造函数是按成员拷贝构造，这导致了两个不同的指针(如ptr1=ptr2)指向了相同的内存。当一个实例销毁时，调用析构函数free(ptr1)释放了这段内存，那么剩下的一个实例的指针ptr2就无效了，在被销毁的时候free(ptr2)就会出现错误了, 这相当于重复释放一块内存两次。这种情况必须显式声明并实现自己的拷贝构造函数，来为新的实例的指针分配新的内存。
### 虚函数
1. 虚函数是动态绑定的，使用虚函数的指针和引用能够正确找到实际类的对应函数，而不是执行定义类的函数。 
2. 构造函数不能是虚函数。而且，在构造函数中调用虚函数，实际执行的是父类的对应函数，因为自己还没有构造好, 多态是被disable的。 
3. 析构函数可以是虚函数，而且，在一个复杂类结构中，这往往是必须的。
4. 将一个函数定义为纯虚函数，实际上是将这个类定义为抽象类，不能实例化对象。 
5. 纯虚函数通常没有定义体，但也完全可以拥有。
6.  析构函数可以是纯虚的，但纯虚析构函数必须有定义体，因为析构函数的调用是在子类中隐含的。 
7. 非纯的虚函数必须有定义体，不然是一个错误。 
8. 派生类的override虚函数定义必须和父类完全一致。除了一个特例，如果父类中返回值是一个指针或引用，子类override时可以返回这个指针（或引用）的派生。例如，在Base中定义了 virtual Base* clone(); 在Derived中可以定义为 virtual Derived* clone()。可以看到，这种放松对于Clone模式是非常有用的。
#### 构造函数为什么不能是虚函数
1. 虚函数对于一个虚函数表，存储在对象的内存空间，如果构造函数时序的。对象没有实习化，无法调用虚函数表，无法实例化。
2. 实现上vbtl在构造函数调用后才建立，因而构造函数不可能成为虚函数  
3. 当一个构造函数被调用时，首先要是初始化VP R。而只能知道它是“当前”类的，而完全忽视这个对象后面是否还有继承者。当编译器为这个构造函数产生代码时，它是为这个类的构造函数产生代码- -既不是为基类，也不是为它的派生类（因为类不知道谁继承它）。
3. 构造函数不需要是虚函数，也不允许是虚函数，因为创建一个对象时总要明确指定对象的类型，尽管可以通过基类的指针或引用去访问它。但析构却不一定，我们往往通过基类的指针来销毁对象。这时候如果析构函数不是虚函数，就不能正确识别对象类型从而不能正确调用析构函数。——基类的析构函数一般都为虚函数。
### 内联函数、静态成员函数不能是虚函数
- inline是编译时展开，是静态行为，而虚函数是动态行为，两者矛盾，而且inline关键字只是给编译器的意见，inline声明的函数却没有inline。
- static属于class自己,其函数的指针存放也不同于一般的成员函数，其无法成为一个对象的虚函数的指针以实现由此带来的动态机制

## STL
### STL六大组件
1. 容器(Containers):数据结构：序列式容器(vector,list,deque),关联式容器(set,map,multiset,multimap),一种class template.
2. 算法(algorithms): 常用算法，如：sort、search、copy、erase。从实现的角度来看，STL算法是一种 function template。注意一个问题：任何的一个STL算法，都需要获得由一对迭代器所标示的区间，用来表示操作范围。这一对迭代器所标示的区间都是前闭后开区间，例如[first, last)
3. 迭代器(iterators): 容器和算法之间的胶合剂，是所谓的"泛型指针",以及其他衍生变化。从实现的角度来看，迭代器是一种将 operator*、operator->、operator++、operator-- 等指针相关操作进行重载的class template。所有STL容器都有自己专属的迭代器，只有容器本身才知道如何遍历自己的元素。原生指针(native pointer)也是一种迭代器。
4. 仿函数(functors)：行为类似函数，可作为算法的某种策略（policy）。从实现的角度来看，仿函数是一种重载了operator（）的class或class template。一般的函数指针也可视为狭义的仿函数。
5. 配接器(adapters)：一种用来修饰容器、仿函数、迭代器接口的东西。例如：STL提供的queue 和 stack，虽然看似容器，但其实只能算是一种容器配接器，因为它们的底部完全借助deque，所有操作都由底层的deque供应。改变 functors接口者，称为function adapter；改变 container 接口者，称为container adapter；改变iterator接口者，称为iterator adapter。
6. 配置器(allocators)：负责空间配置与管理。从实现的角度来看，配置器是一个实现了动态空间配置、空间管理、空间释放的class template。

这六大组件的交互关系：container（容器） 通过 allocator（配置器） 取得数据储存空间，algorithm（算法）通过 iterator（迭代器）存取 container（容器） 内容，functor（仿函数） 可以协助 algorithm（算法） 完成不同的策略变化，adapter（配接器） 可以修饰或套接 functor（仿函数）
### 容器实现原理
- vector：动态数组，是一段连续的空间，访问速度快，初始容量为15，超出容量时以1.2被速度扩展，通过拷贝原来的数组到新分配的数组中，插入删除会造成迭代器失效。
- list： 双向列表，插入删除都不会造成迭代器失效。
- deque：双向的连续线性空间，分配中央控制器map(并非map容器)，map是一小块连续空间，里面每个元素都是一个指针，指向另一段连续性空间（缓冲区：dequeue存储空间的主体）。所以使用deque的复杂度要大于vector，尽量使用vector。
stack、queue:基于deque
- heap-完全二叉树，使用最大堆排序，以数组(vector)的形式存储
- priority_queue-基于heap，底层是vector。
- set,map,multiset,multimap-基于红黑树(RB-tree)，一种加上了额外平衡条件的二叉搜索树。
- hash table-散列表。将待存数据的key经过映射函数变成一个数组(一般是vector)的索引，例如：数据的key%数组的大小＝数组的索引(一般文本通过算法也可以转换为数字)，然后将数据当作此索引的数组元素。有些数据的key经过算法的转换可能是同一个数组的索引值(碰撞问题，可以用线性探测，二次探测来解决)，STL是用开链的方法来解决的，每一个数组的元素维护一个list，他把相同索引值的数据存入一个list，这样当list比较短时执行删除，插入，搜索等算法比较快。
- hash_map,hash_set,hash_multiset,hash_multimap-基于hashtable。
### vector和list区别
- vector拥有一段连续的内存空间，因此支持随机存取，如果需要高效的随即存取，而不在乎插入和删除的效率，使用vector。
- list拥有一段不连续的内存空间，因此不支持随机存取，如果需要大量的插入和删除，而不关心随即存取，则应使用list。
### it++和++it
    ++it直接返回引用，it++会返回一个无用的临时对象，而且因为是函数重载，编译器无法优化，没遍历一遍都要删除一遍无用的临时遍历，所以++it更好。
# c++11新特性
### lambda表达式，匿名函数
'''c++
[capture](parameters)->return-type{body}//可以无参数，即parameters为空，返回值也可以空
[capture](parameters){body}//返回值为空
//例子
vector<int> A = {3,2,1,4};
sort(A.begin(),A.end(),[&](int x, int y)->bool{return x > y;});
'''
　Lambda函数可以引用在它之外声明的变量. 这些变量的集合叫做一个闭包. 闭包被定义在Lambda表达式声明中的方括号[]内. 这个机制允许这些变量被按值或按引用捕获.如下：
```c++
[] //未定义变量，试图在Lambda外使用任何外部变量都是错误的.
[x,&y]//x按值捕获，y按引用捕获
[&] //用户用到的所有外部变量都是隐式按引用捕获
[=] //用户用到的所有变量都是隐式按值捕获
[&,x]//x显式按值捕获，其它变量按引用捕获
```
一个没有指定任何捕获的lambda函数，可以显式的转化成一个具有相同声明显示的函数指针，如：
'''c++
auto a_lambda_func = [](int x){/*...*/};
void (*func_ptr)(int) = a_lambda_func;
func_ptr(4);//调用lambda表达式
'''
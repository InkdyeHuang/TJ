<!-- GFM-TOC -->
* [匿名函数](#匿名函数)
* [numpy和pandas区别](numpy和pandas区别)
<!-- GFM-TOC -->

###匿名函数
语法： lambda arg1,arg2,...argn:expression
```python
f = lambda a,b,c: a+b+c;
print f(1,2,3)
```

###numpy和pandas区别
Numpy是以矩阵为基础的数学计算模块，纯数学。
Scipy基于Numpy，科学计算库，有一些高阶抽象和物理模型，如傅立叶变换。
Pandas提供了一套名为DataFrame的数据结构，比较契合统计分析中的表结构，并且提供了计算接口，可用Numpy或其它方式进行计算,有类似数据库里的SQL操作。

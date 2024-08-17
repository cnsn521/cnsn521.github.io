### 当前安排

1.  先重拾基础。《C++ primer》/《Effective C++》 / 侯接视频。\*\*21年底前完成所有的基础工作。一共两个多月。\*\*C++ primer工具书也再次浏览了一遍，大而广的东西没能够深入，因为没有应用场景，因此这个工具书待到遇到再回来看看。

2.  深入[[开源项目](https://so.csdn.net/so/search?q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&spm=1001.2101.3001.7020)](https://so.csdn.net/so/search?q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&spm=1001.2101.3001.7020)。Redis/Muduo等。将在下年深入进行，这年只是大概看看。

兼顾好实验室项目以及课程的同时，尽量抽时间完成。加油。

* * *

本篇内容
----

本篇记录的是【侯捷 - C++[[面向对象](https://so.csdn.net/so/search?q=%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1&spm=1001.2101.3001.7020)](https://so.csdn.net/so/search?q=%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1&spm=1001.2101.3001.7020)高级开发】入门课程。

本课程一共是上下两部分，上部分主要讲基础的OOP思想以及方法，下部分是深入的解析。总课时估计是十来小时，放在四五天学习会挺舒服的。

* * *

### 侯捷视频资源

源自公众号：编程指北

链接: https://pan.baidu.com/s/19REVrk-\_3lpQu\_fUmRBRUw 密码: 7iup

或有条件也可以直接访问Youtube资源。

* * *

篇2 头文件与类的声明
-----------

### 为什么面向对象

将数据和函数方法绑定在一起，方便（方法整合到了一起）、安全且隐私（数据不被观察）。

### Header防卫式声明

多个文件引入头部文件的时候，避免重复定义。

攥写方法如下：

**complex.h**

```
#ifndef __COMPLEX__ 
#define __COMPLEX__

#endif 
```

### Header布局

**前置声明、类声明、类定义。** 

```
class ostream;
class complex;

class complex{};

complex::function(){

}
```

### 类模板

希望类中某个属性不直接绑定内置类型，因此直接把类型抽象成一个模板。

```
template<typename T>
class complex{
public:
	complex(T a,T b):re(a),im(b){}
private:
	T re,im;
};

complex<double> c2(2.5,1.5)
complex<int> c1(2,6)
```

* * *

篇3 构造函数
-------

### inline 内联函数

如果函数在**class内**定义完成，则会成为inline函数（但要求函数简单，编译器不拒绝）。

但某些复杂函数无法inline，编译器会拒绝。

### access level 访问级别

public / private

建议所有的数据都放在private。

### 构造函数

#### 构造函数的通常写法

构造函数的函数名称与类一致。无返回值。

另外，不带指针的类通常不需要写析构函数。

```
complex(double r = 0,double i = 0):
	re(r), im(i){	}
	

complex(double r = 0,double i = 0)
	{	re = r ; im = i; }
```

#### 支持重载，**但是不可以让编译器产生歧义，即入两个函数都可以（等价可以）**

重载不仅指入参可以不一样，出参也可以不一样。

用作成员set和用作get的函数名可以一致。

```
double real() const{return re;}

void real(double r){re = r;}
```

```
complex(double r = 0,double i = 0):
	re(r), im(i){	}

complex():
	re(0), im(0){	}
```

#### Singleton 单例模式

把构造函数放在private，就不能被公共创造。

```
class A{
public:
	static A& getInstance();
	setup(){...}
private:
	A(); 
	A(const A& rhs); 
}

A& A::getInstance(){
	static A a;
	return a;
}
```

* * *

篇4 参数传递与返回值
-----------

### const member functions 常量成员函数

定义函数的时候，不希望该函数有修改成员值的权力，就加const限定词。

```
double real() const{ reutrn re; }
```

### 参数传递：pass by value vs. pass by reference(to const)

尽量传ref。

传value是复制了一份数据到栈上。

传ref是直接传引用（引用的地步就是指针，四个字节）。

所以传ref快，**而且方便改值**（尽管传char可能更小，一个字节，但是没必要这么细，通常来说传ref快）。当然如果希望不被改值，加const关键词。

```
complex& operator += (const complex&);
```

### 返回值传递：return by value vs. return by reference(to const)

**尽量返ref。** 

### friend 友元

通常封装数据之后，外部都无法访问数据。\*\*但是友元可以。\*\*事实上友元有点违反封装的设计原理。

```
class complex{
    
private:
    
    friend complex& __doapl(complex*,const complex& r);
}

inline complex&
    __doapl(complex* ths, const complex& r){
    ths->re += r.re; 
    this->im += r.im;
    return *ths; 
}
```

#### 相同class的各个objects互为friends

相同class的各个objects互为friends。即同类的objects可以互相访问private data。

* * *

篇5 操作符重载 operator overloading
-----------------------------

### 成员函数版本 含this

**因此需要在类中写好定义。** 

使用场景

```
complex c1;
complex c2;
c1 += c2;

complex c3;
c3 += c2 += c1;
```

则实现方式：

```
inline complex& complex::operator += (const complex& r){
	return __doapl(this,r);
}
```

这里隐藏了操作符的左操作数，是this，参数列表中不需要显式写出，可以直接使用。

### 非成员函数版本 不含this

#### 二元加法

使用场景

```
complex c1;
complex c2;

c2 = c1 + c2;
c2 = c1 + 5;
c2 = 7 + c1;
```

则实现方式：

这个绝对不可以返回reference，因为相加的object一定是新建且是local的。因此需要创建临时对象（无名称，只在当前行有用）返回。`临时对象的内存地址与return出去后的内存地址一致吗？`

#### 一元正负

使用场景

```
complex c1;
+c1;
-c1; 
```

实现方法

```
inline complex&
operator +(const complex& x){
	return x;
}

inline complex&
operator -(const complex& x){
	return complex(-real(x),-imag(x));
}
```

#### 二元判断是否相等，是否不等

使用场景

```
complex c1;
complex c2;

c1 == c2;
c1 == 0;
0 == c1; 
```

##### 比较相等

![](https://i-blog.csdnimg.cn/blog_migrate/02f8ade0684334ee66bd0a85b69e558d.png)

##### 比较不等

![](https://i-blog.csdnimg.cn/blog_migrate/d11c615e0ad3fff8a92c18711713243b.png)

#### <<重载（std重载）

使用场景

```
complex c1(2,1);
cout << c1; 
cout << c1 << c1; 
```

实现方案

```
#include <iostream>

ostream&
operator << (ostream& os, const compelex& x){
    return os << '(' << real(x) <<',' << imag(x) << ')';
}
```

**以上是经典案例1，complex，class without pointer member**

**以上是经典案例2，string，class with pointer member**

* * *

篇7 Big Three：拷贝构造、拷贝赋值、析构
-------------------------

场景

```
String s1();
String s2("我是一个酒精过敏的帅哥");
String s3(s1); 
s3 = s2; 
```

### Big Three

#### 类声明

\*\*如果成员函数含有指针，必须含有copy ctor和copy op=。\*_否则如果使用默认的方法，只是潜赋值，会共用同一个底层的char_。

```
class String {
public:
    String (const char* cstr = 0); 
    String (const String& str); 
    String& operator=(const String& s); 
    ~String(); 
    char* get_c_str() const(return m_data); 
private:
    char* m_data;
}
```

#### constructor和destructor定义

```
inline 
String::String(const char* cstr = 0){
    if (cstr){
        m_data = new char[strlen(cstr)+1];
        strcpy(m_data,cstr);
    }
    else{ 
        m_data = new char[1];
        *m_data = '\0';
    }
}
inline 
String::~String(){
    delete[] m_data;
} 
```

#### copy ctor（拷贝构造）

```
inline 
String::String(const String& str){
    m_data = new char[str(str.m_data) + 1];
    strcpy(m_data,str.m_data); 
 }
```

#### copy assignment operator(拷贝赋值函数)

```
inline
String& String::operator=(const String& str){
    if (this == &str){ 
        return *this;
    }
    delete[] m_data; 
    m_data = new char[ strlen(str.m_data) + 1]; 
    strcpy(m_data,str.m_data); 
    return *this;
} 
```

篇8 栈stack、堆heap与内存管理
--------------------

### 堆栈说明

*   栈，存在于某个作用域。
*   堆，全局内存空间，可动态分配内存。

```
{
    Complex c1(1,2); 
    Complex* p = new Complex(3); 
} 
```

### static local objects的生命周期

其生命在作用域结束后仍然存在

```
{
	static Complex c;
} 
```

### global objects的生命周期

其生命一直存在，直到程序结束

```
Complex c3; 
int main{

}
```

### heap objects的生命周期

存在，直到被delete掉。如果不delete则内存泄漏（即没有回收，且没有办法再手动回收）。

```
{
	Complex* p = new Complex;
	
	delete p;
}
```

#### new的过程

1.  **先分配内存**：使用malloc函数，大小为sizeof(对象大小)，对象大小是成员变量决定的，（与成员函数无关）。
    1.  如果创建单个对象：如Complex两个double，则8字节，**头尾要加上cookie，表明该部分内存已经被占用，一个cookie为4个字节。即release下8+2\*4 = 16字节（必须是16字节的倍数，如果不足则向上取整）。debug下就得再加32+4，其实没必要记。** 
    2.  如果创建对象数组：除了要n份成员变量以外，需要额外一个字节来记录数组的大小。\*\*即如创建长度为3的Complex数组，release下：8 \* 3（成员变量） +4 \* 2 （cookie）+ 4 (记录长度) = 36 -> 48 \*\*
2.  再调用构造函数

#### delete的过程

1.  先**调用析构函数**
2.  再**释放内存**：

* * *

篇10 类 拓展：模板、函数模板
----------------

### 补充1：static

**全局空间**会有一份类中的方法、静态成员变量、静态成员函数。而正常的成员变量会在初始化object的时候创建。

1.  普通函数处理普通数据（需要传this pointer），当然也可以处理静态数据。

2.  静态函数不能处理普通数据，**只能**处理静态数据。

3.  静态数据除了在类里面做好生命，必须要在类外做好**定义**。

4.  Singleton中用到了静态变量(唯一的实例)、静态函数（get方法）。

    1.  优化方案：去掉静态类变量，放在静态函数里面；这样只有有人调用get方法的时候，实例才会被创建。

```
class Account{
publid:
    static double m_rate;
	static void set_rate(const double& x){m_rate = x;}
}
double Account::m_rate = 8.0;

int main)_{
    Account::set_rate(5); 
    Account().set_rate(5); 
} 
```

### 为什么cout可以接受那么多类型去输出

cout继承于ostream，标准库团队在ostream实现重载了大量类型的<<操作符。

### function template，函数模板

使用场景：任意元素的比较大小，使用函数模板

```
template <class T>
    inline 
    const T& min(const T&a,const T&b ){
    	return b<a ? b:a;
} 
```

当然如果自己设计的类要调用该函数，一定是需要自己重载<操作符。

```
class A{
public:
    bool operator < (const stone& rhs) cosnt{
        return this.___< ths.___;
    }
}
```

### namespace 命名空间

namespace是一个区域限制，相当于局部空间。可以如下使用：

1.  using directive

```
using namespace directive
```

2.  using declaration

```
using std::cout;
cout<<"hello";

std::cout<<"hello"; 
```

* * *

篇11 组合、委托与继承
------------

面向对象的三个重要特性：复合、委托、继承。

### Composition 复合, has-a

一个类中，包含了另一个类，就叫复合。生命周期同步。

如下，queue类里面包含了Sequence类。

![](https://i-blog.csdnimg.cn/blog_migrate/5e227ca2cb3b9305395cecfebada98ce.png)

```
template<class T, class Sequence = deque<T>>
class queue{
    
protected:
    Sequence c;
}
```

#### 构造与析构的关系

假如Container has a Component.

1.  **构造由内而外。**Container的构造西首先调用Component的**default** constructor，然后才执行自己的ctor。如果不想用deault，则需要自己构造Component.
2.  **析构由外而内。** 

### Delegation 委托，Composition by reference

一个类，仍然包含另一个类，但是不是通过内存直接包含，而是用一个指针包含。 **生命周期不同步。** 

如下，String类委托了一个StringRep类。

![](https://i-blog.csdnimg.cn/blog_migrate/1957318a0fcba379677ae34c5baf4912.png)

```
class StringRep;
class String{
    
private:
    StringRep* rep; 
}
```

委托可以对外接口一致，String接口永远不变，但是内部实现可以通过修改StringRep改变。

### Inheritance 继承，is-a

类A是类B，则类A继承类B。这个关系清晰易懂，用显示情况get![](https://i-blog.csdnimg.cn/blog_migrate/fa24f72ed4ccbbfb9899067a9cf3eea9.png)

#### 构造和析构

1.  \*\*构造由内而外。\*\*Derived的构造函数首先调用Base的default构造函数，然后才执行自己。
2.  **析构由外而内。** 

#### 继承 with 虚函数 virtual functions

##### 虚函数

*   虚函数：如果希望子类重新定义，则可以把函数声明为虚函数，但同时你需要自己写一个默认定义。
*   纯虚函数：自己不给默认，希望子类一定去定义。

```
class Shape{
public:
	virtual void draw() const = 0; 
    vitual void error(const std::string& msg); 
    int objectID() const; 
} 
```

### 委托+继承 经典案例

#### 经典案例1（设计模式Observer）

课程说了一个经典案例，多个Obsever观察同一个数据/文档。每个Observer都有自己对数据的显示方式。

![](https://i-blog.csdnimg.cn/blog_migrate/b611defebf1809afe734e2b50b076670.png)

代码如下：

```
class Subject{
   int m_value; 
   vector<Observer*> m_views; 
public:
   void attach(Observer* obs){
       m_views.push_back(obs);
   }
   
   
   void set_val(int value){
       m_value = value;
       notify();
   }
   
   void notify(){
       for(int i=0;i<m_views.size();++i)
           m_views[i]->update(this,m_value); 
   }
}

class Observer{
public:
   virtual void update(Subject* sub,int value) = 0; 
} 
```

#### 经典案例1（设计模式Composite）

尚未感受到有多厉害，截个图把。

![](https://i-blog.csdnimg.cn/blog_migrate/335efebe6ecc6343b2a4c862757657f7.png)

#### 经典案例2（设计模式Prototype）

**父类**想创建**未来**才定义的子类。同样还没感受到，截个图然后教程继续。

![](https://i-blog.csdnimg.cn/blog_migrate/8ec38b2b09f5b4787c02bf2e55458830.png)

* * *

承上启下：兼谈对象模型
-----------

后面是“面向对象程序设计”的续集。会讨论：

*   泛型编程、面向对象编程
*   继承的深处：this指针，虚指针，虚表，虚机制。

即讲述以下技术点

![](https://i-blog.csdnimg.cn/blog_migrate/740059216f55e9ca5ebda551658bb391.png)

* * *

篇2 conversion function，转换函数
---------------------------

使用场景：我自定义一个类，表示分数，但是我希望它做加减乘除的时候，自动转为double。

```
class Fraction{
public:
    ...
    operator double() const{ 
        return (double)(分子/分母);
    }
}

Fraction my_frac(3,5);
double d = 4 + my_frac;
```

#### 非explicit单实参构造函数 non-explicit-one-argument ctor

```
class Fraction{
public:
    Fraction(int num,int den=1):分子(num),分母(den){};
    
    Fraction operator+(const Fraction& f){
        return Fraction(...)
    }
}
```

我们的使用场景仍然和上面一样，希望`Fraction d = 4 + my_frac;` ，只需要把4隐式构造成Fraction，然后用操作符+相加起来。

设计模式里面的【代理】，就用到了转换函数。

篇4 指针类 pointer-like classes
---------------------------

像指针的类，用起来跟指针一样，但是有更多的机制。

链表的node也是一种pointer-like classes。真挺厉害的。

![](https://i-blog.csdnimg.cn/blog_migrate/dac549e639c9d036fe3e5310659bcd2e.png)

以下给出node的代码

```
T& reference operator*()const{
    return (*node).data; 
}

T* pointer operator->() const{
    return &(operator*()); 
}

list<Foo>::iterator ite;
*ite; 
ite->method();
```

* * *

篇5 仿函数 function-like classes
----------------------------

让一个类像函数被调用，只要实现()操作符即可。

小小的类，用作base unit。

* * *

篇6 namespace经验谈
---------------

把区域分隔开。用`::`访问特定区域。

* * *

篇9 member template，成员模板
-----------------------

使用场景：更灵活地构造

```
pair<Derived1,Derived2> p;
pair<Base1,Base2>p2(p); //支持这种做法，就必须些成员模板。只能up-cast 
```

![](https://i-blog.csdnimg.cn/blog_migrate/2528ea9576e7cd48e3c2c7ae9d66de35.png)

同时智能指针也实现了成员模板。

* * *

篇10 Specialization，模板特化
-----------------------

泛化模板覆盖很广，但是我们有时候需要针对某种类型，执行不同的操作。

```
template<class Key>
    struct hash{};

template<>
struct hash<int>{
    size_t operator()(int x) cosnt{return x;}
} 
```

* * *

篇11 partial specialization，模板偏特化
--------------------------------

这部分跟python的偏函数挺像的，就是对template泛化模板，偏特化某个模板参数 或者 修改范围。

个数上的偏特化

```
tempalte<typenamte T, typename Alloc=...>
    class vector{
      ...  
    };

tempalte<typename Alloc=...>{
	...    
}
```

范围上的偏特化

```
temaplte<typename T>
class C{
	...
};

template<typename U>
class C<T*>{
    ...
};

c<string> obj1; 
c<string*> obj2;
```

* * *

篇12 template template parameter，模板模板参数
--------------------------------------

使用场景：希望自定义一个模板，嵌套另一个模板进行使用。

这里有点深了，先有个印象然后跳过吧。

```
template <typename T,
		template<typename T> class Container
          >
class XCls{
private:
    Container<t> c;
...
}
   
//使用
template<typename T>
using Lst = list<T,allocator<T>>;
Xcls<string,Lst>mylst2;
```

* * *

篇13 如何确认当前cpp版本
---------------

`cout<<__cpluscplus;` 如果输出201103，则表示是C++11。

* * *

篇14 c++的三个主题：可变参数
-----------------

### variadic templates，可变模板

使用场景：使某函数接受任何多的参数。

```
void print(){
    
}

template<typename T,typename... Types>
void print(const T& firstArg,const Types&... args){
    cout<<firstArg<<endl;
    cout<<sizeof...(args)<<endl; //这里就知道参数包的size大小
    print(args...); //递归调用
}

//使用
print(7.5,"hello",bitset<16>(377),42);
```

### auto

让编译器自动推导type。

```
list<string> c;

list<string>::iterator ite;
ite = find(c.begin(),c.end(),target);

auto ite = find(c.begin(),c.enmd(),target); 
```

### ranged-base for

新的循环方式，像Python一样。

```
for (int i:{2,3,4,7}){
    cout<<i<<endl;
}

vector<double> vec;
for(const auto& elem:vec){ 
    cout<<elem<<endl;
}
```

篇15 reference 引用详解
------------------

特性：

1.  sizeof® = sizeof(x)，即引用大小与原来的**元素大小一样，值也一样，地址也**一样。（这是编译器做的封装，底层肯定是不一样的）。

```
int x = 0;
int* p = &x; 
int& r = x; 

int x2 = 5;
r = x2; 
int& r2 = r; 
```

2.  引用比指针要优雅，而且可以使得接口调用的时候，保持一致（无论是否引用，都一致）。

```
void f2(Cls obj){obj.xxx();}
void f3(Cls& objs){obj.xxx();} 

f2(obj);
f3(obj);
```

**另外，函数的const是否属于函数签名一部分呢？是的。** 

* * *

篇17 关于vptr和vtbl
---------------

[[高清 1080P C++面向对象高级编程（侯捷） P30 17 关于vptr和vtbl](https://www.youtube.com/watch?v=MysC48nh9yI&list=PL-X74YXt4LVZ137kKM5dNfCIC4tsScerb&index=29)](https://www.youtube.com/watch?v=MysC48nh9yI&list=PL-X74YXt4LVZ137kKM5dNfCIC4tsScerb&index=29)

只要类中有虚函数，就有vptr，它指向vtbl，表里面放的都是函数指针，指向内存里面的虚函数。

这幅图画得非常非常清晰。

![](https://i-blog.csdnimg.cn/blog_migrate/566e00c7e1c7f68100fb54307d8456a0.png)

这个也就是实现**多态**的底层原理。

* * *

篇18 动态绑定
--------

### 关于静态绑定和动态绑定

**静态绑定**直接访问内存里编译好的函数内存空间。

**动态绑定**调用三个条件：

1.  必须通过指针调用。**（this指针调用即可满足）**
2.  必须是up-cast（子类向父类转型）**（调用父类的函数即可满足）**
3.  必须调用的是虚函数 **（父类需要动态绑定的函数定义为虚函数）**

### 底层勘探

假如A是B的父类，B是C的父类。

```
B b;
A a= (A)b; 
a.vfunc1(); 


A* pa = new B; 
pa->vfunc1(); 
```

篇19 谈谈const
-----------

这里Cpp primer这本书说得贼全，但是很复杂。侯老师说得比较精炼。

当成员函数的const和non-const版本同时存在，  
**const object** 智能调用**const**版本。  
**non-const object**只能调用**non-const**版本。

|  | const object  
(data member不得改动) | non-const object  
（data members可改动） |
| --- | --- | --- |
| const member functions  
**(保证不改变data members)** | 允许 | 允许 |
| non-const member functions  
**(不保证 dat members不变)** | 不允许 | 允许 |

即**常量对象无法调用非常量成员函数。** 

#### COW: Copy and Write

当重载\[\]操作符的时候，设计两个函数，

1.  const member function不必考虑COW（访问同一个内存空间的object）
2.  但是non-const member function是必须考虑COW（申请一个新的内存空间，copy过去）。
3.  当成员函数的const和non-const同时存在的时候，C++特性：const object智能调用const版本，non-const object只能调用non-const版本。

* * *

后话
--

恭喜完成了【侯捷 C++ 面向对象高级开发】的学习！  
本课程一共是上下两部分，上部分主要讲基础的OOP思想以及方法，下部分是深入的解析。总课时估计是十来小时，放在四五天学习会挺舒服的。

Cpp是一个庞然大物，不必期待那么快就能学完。

后面我将继续进行侯接课程的学习，持续更新。

欢迎评论交流！
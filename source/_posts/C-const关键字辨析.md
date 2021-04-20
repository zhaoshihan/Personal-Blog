---
title: C++ const关键字辨析
mathjax: true
categories: C/C++
abbrlink: 35148
date: 2021-02-22 16:57:45
tags:
---

最近学习C/C++语言，又一次看到了`const`关键字。之前本科的时候这个地方就一直没有搞的很明白，这次我想专门写篇文章来帮助自己记忆。以下的内容参考《C++ Primer》（第五版）



首先，我们要明确：C++语言的类型（Type）分为两种：一种叫做普通类型（Primitive Build-in Type）；一种叫做复合类型（Compound Type），const关键字对于这两种类型的作用效果是不一样的



由于C++的编译过程是各个文件分开的（separate compilation），所以需要一点概念辨析：

**变量（Variable）就是有名称（name）的对象（Object）；一个Object就是有类型（type）的一定量的内存空间**

什么是变量声明（declaration）？就是明确指出变量的类型（Type）和名称（name）

```C++
extern int i; // declares, but does not define i
# 这里的变量i一般是通过#include引用的来自其他文件中定义的变量
# 一般情况下，一个变量可以声明多次，但是只能定义一次，最常见的例子就是std::cout, std::cin（#include <iostream>），它在<iostream>文件中被定义，但是可以被各个文件include并声明使用
```

什么是变量定义（definition）？就是明确指出变量的类型（Type）和名称（name），同时分配内存空间，有可能伴随着变量的初始化

```C++
int j; // declares and defines j
int num = 0; // declares, define, initialize

extern double pi = 3.1416 // definition
# 当声明（declaration）伴随着显式地初始化（initialize）时，同样也是一种变量的定义（definition）
# 此时，相当于覆盖了extern关键字的意义（override）
```

**一个变量可以声明多次，但是只能定义一次**

什么是变量初始化（initialize）？就是说变量在创建（created，分配了内存空间）的同时获得指定的值（specified value）

```C++
int units_sold = 0;
int units_sold(0);

# new standard
int units_sold = {0};
int units_sold{0};
```

对于我们自定义的Class Type，变量在没有显式声明的情况下会进行默认初始化（default initialized）

```C++
std::string str; // str implicitly initialized to the empty string, 这是由std::string class定义的
Sales_item item; // default-initialized Sales_item object
```

对于Primitive Build-in Type，分为两种情况来说：如果这个build-in type的变量是在所有函数体之外定义（define）的，那么会被默认初始化为0；如果这个build-in type的变量是在函数体内部被定义的，那么值就是`undefined`

什么是变量赋值（assignment）？就是说将变量当前的值（current value）擦除，替换成新的值，通常使用`=`



**在C++中，初始化（initialize）和赋值（assign）是完全不同的两个概念**



## Const Primitive Type

普通类型（Primitive Build-in Type）包括了以下这些：

|    Type     |              Meaning              |     Minimum Size      |
| :---------: | :-------------------------------: | :-------------------: |
|    bool     |              Boolean              |          NA           |
|    char     |             Character             |        8 bits         |
|   wchar_t   |          Wide character           |        16 bits        |
|  char16_t   |         Unicode character         |        16 bits        |
|  char32_t   |         Unicode character         |        32 bits        |
|    short    |           short integer           |        16 bits        |
|     int     |              Integer              |        16 bits        |
|    long     |           long integer            |        32 bits        |
|  long long  |           long integer            |        64 bits        |
|    float    |  Single-precision floating-point  | 6 significant digits  |
|   double    |  Double-precision floating-point  | 10 significant digits |
| long double | Extended-precision floating-point | 10 significant digits |

作用于普通类型上的`const`关键字很容易理解，就是说这个类型的Object的值不能改变（不能被反复多次赋值，assignment）。因此我们规定，一个`const`的object在定义（define）的时候就必须完成初始化（initialized），并且在以后的生命过程中不能被赋值（assignment）

```C++
const int i = 42; // ok, initialized at compile time
const int j = get_size(); // ok, initialized at run time
const int k; // error, k is uninitialized const
....
i = 100; // 对于const的object进行赋值操作是错误的
```

```C++
int i = 42;
const int ci = i; // ok, the value in i is copied into ci
int j = ci;	// ok, the value in ci is copied into j
```



## Const Reference Type

复合类型（Compound Type）主要包括了两种，一种是引用（Reference）；一类种是指针（Pointer）。我们这里先考虑`const`关键字对于引用的影响。

首先，我们要明确一点：引用（Reference）不是一种Object（因此也就没有内存地址，不能通过`&`取地址符号获得内存地址），它只是某个Object的一个别名（Alias）。不管前面有没有加`const`关键字，引用本身都具有以下几个特点：

- 引用在定义的同时就必须要初始化

  ```C++
  int ival = 1024;
  int &refVal = ival; // refVal refers to (is another name for) ival
  int &refVal2; // error, a reference must be initialized
  ```

- 引用在经过初始化之后，就不会再绑定（rebind）其他的Object

- 当完成初始化后，所有对引用的操作就是对引用所绑定的那个Object的操作

- 因为引用需要绑定Object，而引用自己本身不是Object，所以不存在多重引用（reference that refer to a reference）

在以上的情况下，我们来说明`const`关键字对引用变量的影响：

在引用定义/初始化时，在Type前面加入`const`（reference to const），**它表示在之后的程序中，不能通过这个引用来修改（这个引用所绑定的）Object的值**

```C++
const int ci = 1024;
const int &r1 = ci; // ok, both reference and underlying object are const
r1 = 42; // error, r1 is a reference to const
int &r2 = ci; // error, non const reference to a const object
```

因此，第一，对于一个本身已经是`const`的Object来说，不能使用没有（前面加）`const`的引用类型来绑定；第二，**尽管不能通过这个引用来修改绑定的Object，但是可以通过其他的方式修改Object的值**（所以说，`const`Type的引用可以绑定非`const`类型的Object）

```C++
int i = 42;
const int &r1 = i; // we can bind a const int& to a plain int object
int &r2 = i; // also okay
r1 = 0; // error, r1 is a reference to const
i = 0; // ok, 直接通过赋值改变Object的值
r2 = 0; // ok, r2 is not const, i is now 0, 或者通过其他引用来修改Object的值
```



## Const Pointer Type

首先，先说指针本身，它自己也是一个Object，指针里存储的值是它所指向的那个Object的地址（address）

- 指针在定义时不一定要初始化
- 指针存储的是它指向/绑定的那个Object的地址
- 指针在完成初始化之后依然可以绑定赋值（指向）为其他的Object
- 对指针来说，解引用操作意味着返回指针指向的Object

接下来考虑最复杂的情况，`const`关键字对于指针的影响

首先，和引用类型一样，指针可以被定义为指向const Object的指针。此时，**同样不会保证这个指针指向的Object不会发生改变，只是能保证不能通过这个指针来改变Object的值**

```C++
double num = 1.0;
const double pi = 3.14;
double *ptr = &pi; // error, ptr is a plain pointer
const double *cptr = &pi; // ok, cptr may point to a double that is const
cptr = &num; // 同样是可以的，这是因为对于指针cptr来说，const属于low-level, low-level non-const可以赋值给low-level const
*cptr = 42; // error, cannot assign to *cptr
```

此外，指针由于自己本身也是一个Object，因此可以定义自身是`const`的指针，此时，这个`const`的指针**必须在定义时完成初始化，并且一旦完成初始化后，指针的值（指向的Object的地址）将不能发生改变**

```C++
int errNumb = 0;
int *const curErr = &errNumb; // curErr will always point to errNumb
const double pi = 3.14159;
const double *const pip = &pi; // pip is a const pointer to a const Object(double), 这里对于声明，需要从右往左来读
```

如果一个Object自身被定义为`const`，那么我们称这种`const`为**top-level const**; 如果一个指针/引用指向/绑定了一个`const`的Object，那么我们称这样的`const`为**low-level const**

对于赋值/拷贝操作来说（`=`），因为拷贝一个Object不会改变被拷贝的那个Object，所以`=`左右两边的Object（左边：被赋值的Object；右边：被拷贝的Object）是否是**top-level const**无关紧要

但是，对于**low-level const**来说就不同了，`=`两边的Object要么具有相同的low-level const（要么都声明为const，要么都不是const）；或者也可以把不是`const`的向`const`转化。但是，不能把`const`赋值给不是`const`的变量

对于引用来说，由于天然地自带了top-level const，因此是否定义为`const`其实就是是否定义为low-level const


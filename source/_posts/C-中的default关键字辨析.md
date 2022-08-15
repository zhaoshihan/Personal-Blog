---
title: C++中的default关键字辨析
mathjax: true
abbrlink: 58985
date: 2022-03-27 16:05:31
categories:
tags:
---


本文旨在说清楚一个问题，就是当我们用C++写下`default`时，到底发生了什么。

首先，`default`关键字是我们告诉编译器（Compiler）我们需要一个自动生成（synthesized）的特殊成员函数（Special member functions），这类的特殊成员函数包括以下几种：

1. Default constructor
2. Copy constructor
3. Move constructor
4. Copy assignment operator
5. Move assignment operator
6. Destructor

接下来所有的情况，都会围绕以上这6个函数来展开，我们依次来说`default`关键字的作用效果。

## Default Constructor

Default constructor的定义是参数列表为空的构造函数（constructor）

如果class中的data memeber具有in-class initializer, 那么就使用它来初始化（initialize）这个member

否则，就使用default-initialize来初始化member

简而言之就是Synthesized Default Constructor函数体为空

```C++
class Sales_data {
  public:
  	Sales_data() = default;
  private:
  	std::string bookNo;	// default initialize
  	int units_sold = 0; // in-class initializer
  	double revenue = 0.0; 
};

// default constructor = default的等价形式
Sales_data::Sales_data() {}
```

## Copy/Move Constructor

按照class中data member在class definition body中的declaration顺序进行一个一个的初始化，初始化的方式是从给定参数object中copy / move member

```C++
class Sales_data {
  public:
  	Sales_data(const Sales_data &orig) = default;
  	Sales_data(Sales_data &&orig) = default;
  private:
  	std::string bookNo;
  	int units_sold = 0;
  	double revenue = 0.0;
};

// copy constructor = default的等价形式
Sales_data::Sales_data(const Sales_data &orig):
	bookNo(orig.bookNo),
	units_sold(orig.units_sold),
	revenue(orig.revenue) {}

// move constructor = default的等价形式
Sales_data::Sales_data(Sales_data &&orig):
	bookNo(std::move(orig.bookNo)),
	units_sold(std::move(orig.units_sold)),
	revenue(std::move(orig.revenue)) {}
// 注意这里即使是右值引用Sales_data &&orig, orig本身作为引用（reference）仍然是一个左值，同样的，它的member(比如orig.bookNo)也默认是左值，因此需要std::move()作转换
// 在data member type声明move constructor = delete的情况下，move constructor = default退化为copy constructor = default的形式
// 在data member type没有default或implicit move constructor的情况下，move constructor = default可以变为以下形式
Sales_data::Sales_data(Sales_data &&orig):
	bookNo(orig.bookNo),
	units_sold(std::move(orig.units_sold)),
	revenue(std::move(orig.revenue)) {}
```

这里要注意的是，`default`只能保证尽量将每个data member进行move constructor（右值）进行初始化。如果说有一个data member的type声明move constructor = delete，那么所有的data member都会变成使用copy constructor（左值）进行初始化；如果说有一个data member的type只声明了copy constructor，那么编译器不会implicit生成move constructor，因此该data member将使用copy constructor（左值）进行初始化，而其他的data member不受影响，继续使用move constructor（右值）进行初始化。

## Copy/Move Assignment Operator

对于class中的data member，依次用right-hand object中的member对left-hand object中的member进行拷贝赋值操作（调用member的copy assignment operator）/ 移动赋值操作（调用member的move assignment operator）

```c++
class Sales_data {
  public:
  	Sales_data& operator=(const Sales_data &rhs) = default;
  	Sales_date& 
};

// copy assignment operator = default的等价形式
Sales_data& Sales_data::operator=(const Sales_data &rhs) {
  bookNo = rhs.bookNo;
  units_sold = rhs.units_sold;
  revenue = rhs.revenue;
  return *this;
}

// move assignment operator = default的等价形式
Sales_data& Sales_data::operator=(Sales_data &&rhs) {
  bookNo = std::move(rhs.bookNo);
  units_sold = std::move(rhs.units_sold);
  revenue = std::move(rhs.revenue);
  return *this;
}
```

## Destructor

Synthesized Default Destructor的函数体为空

```c++
class Sales_data {
  public:
  	~Sales_data() = default;
};

// destructor = default的等价形式
Sales_data::~Sales_data() {}
```

另外，想说一点小细节就是，当我们在class body内的member declaration中使用`=default`，那么自动生成的函数是`inline`的；如果不希望变成`inline` function，就需要在member definition中使用`=default`

## 参考资料

- [cppreference special member function](https://en.cppreference.com/w/cpp/language/member_functions#Special_member_functions)

- [[StackOverflow] Is a `=default` move constructor equivalent to a member-wise move constructor?](https://stackoverflow.com/questions/18290523/is-a-default-move-constructor-equivalent-to-a-member-wise-move-constructor)


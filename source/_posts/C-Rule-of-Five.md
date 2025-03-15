---
title: C++ Rule of Five
mathjax: true
abbrlink: 36014
date: 2022-08-16 00:35:30
categories:
tags:
---

这篇文章的起因是我最近写Leetcode上的设计类题目比较多，因此对C++的 `The Rule of Five` 产生了兴趣，想以此来确定一个通用的代码模板，规范自己今后的所有自定义Class的书写。



## The Rule of Three

`The Rule of Three`指的是一个Class的destructor, copy constructor, copy assignment operator。通常情况下，这三者是在一起出现的：也就是说，如果我们需要定义（define）其中的一个，那么往往意味着我们需要同时定义这三个member function。对于这一情况，《C++ Primier》中有如下的描述：

> Class that need destructor need copy and assignment

当我们的Class涉及内存资源管理时（一般是存在`new`动态分配内存的情况时），**使用Compiler生成的（`default`关键字）三个member function往往是错误的**，因为这种自动生成的member function只会对指针member进行浅拷贝，而不是将实际资源复制一份。



## The Rule of Five

`The Rule of Five`指的是一个Class的destructor, copy constructor, copy assignment operator(这三者又被称为`The Rule of Three`), 外加 move constructor, move assignment operator。这五个member function又被称作为`copy-control` member。

如果我们显式地定义了（或者是`= default`, `= delete`这样的声明）任何的一个`copy-control`  member，那么就会阻止Compiler为我们生成（synthesize）默认的move constructor 和 move assignment operator。如果我们希望获得性能上的优化，那么同样需要自己显式地定义这两个member function。需要注意的是，与`The Rule of Three`不同，**如果我们不提供move constructor, move assignment operator的定义，那么不会造成错误，只是会丧失优化的机会。**

如果我们就是不提供move constructor和move assignment operator, Compiler也不进行自动生成，那么Compiler会把Rvalue reference当作一般的reference来看待，此时就会调用copy constructor和copy assignment operator来处理，极大可能会造成多余的内存空间申请和元素拷贝。



## The Rule of Zero

如果我们的Class中所有的member都遵循了`The Rule of Five`时，那么这个Outside Class就不需要再自定义任何的`copy-control` member，相当于说完成了一种封装，最典型的例子如下：

```c++
class rule_of_zero {
    std::string cppstring;
public:
    rule_of_zero(const std::string& arg) : cppstring(arg) {}
};
```



## Template 

下面给出标准的自定义Class的书写模版，以后凡是涉及`The Rule of Five`的情况都可以依照该模板来写：

```c++
class MyClass {
  public:
    // default constructor
    MyClass(std::size_t size = 0): 
      size_(size), arr_(size_ ? new int[size_] : nullptr) {}
      
    MyClass(const MyClass &other): 
      size_(other.size_), arr_(size_ ? new int[size_] : nullptr) {
      // use deep copy
      std::copy(other.arr_, other.arr_ + other.size_, arr_);
    }
    
    ~MyClass() {
      delete[] arr_;
    }
    
    // use default constructor to initialize this first(constructor delegation)
    MyClass(MyClass &&other): MyClass() {
      swap(*this, other);
    }
    
    MyClass& operator=(MyClass other) {
      swap(*this, other);
      return *this;
    }
    
    friend void swap(MyClass &lhs, MyClass &rhs) {
      using std::swap; // allow use of std::swap...
      swap(lhs.size_, rhs.size_); // ...but select overloads first
      swap(lhs.arr_, rhs.arr_);
    }
  	
  private:
    std::size_t size_;
    int *arr_;
};
```

对于这个模板，我想来着重解释一下几个细节：

首先是`operator=`：这里只是用一个`MyClass& operator=(MyClass other)`来代替copy assignment operator`MyClass& operator(const MyClass &other)` 和 move assignment operator `MyClass& operator=(MyClass &&other)`。由于传入的参数是值而非引用，因此会相应地隐式调用copy constructor或move constructor，生成`MyClass other`

其次是`friend void swap`：这里我们自定义的`swap`函数不应该被视为`MyClass`中的一个member function，而是定义在当前namespace中的一个独立函数。此时的`friend`函数是可以被`ADL`机制发现的。

> A friend function defined inside a class is:
>
> - placed in the enclosing namespace
> - automatically `inline`
> - able to refer to static members of the class without further qualification

然后是`using std::swap`： 引入标准库中的`std::swap`函数作为后备。对于没有`namespace`前缀的函数，C++在函数调用时遵循ADL（Argument- dependent lookup） 的查找规则。我们这里只需要知道，ADL使得我们可以使用在函数实参（argument）类型的namespace中定义的同名函数（在这里就是我们`member variable` class自己定义的`friend void swap`）。如果以上的查找都找不到名为`swap`的函数，那么最后使用我们引入的标准库中的`std::swap`。

最后是`noexcept`： 对于move constructor和move assignment operator，最好是可以声明为`noexcept`，这主要是为了优化上的考量，例如`std::vector<T>`中的`push_back`函数，只有在保证`T` 有`noexcept`声明的move constructor，move assignment operator时，`push_back`函数才会在执行时使用move操作来优化；如果不能保证`noexcept`，那么只能退化为调用copy constructor， copy assignment operator的操作了。



## Copy-and-Swap Idiom

 在C++03时，对于`std::swap`，它使用的是copy constructor和copy assignment operator， 其内部的实现可以近似等价于：

```c++
template<typename T> void swap(T& t1, T& t2) {
  T tmp(t1);
  t1=t2;
  t2=tmp;
}
```

在C++11以后，对于`std::swap`，它使用的是move constructor和move assignment operator，要求`T`必须是`MoveConstructible`和`MoveAssignable`的，其内部实现可以近似等价于：

```c++
template<typename T> void swap(T& t1, T& t2) {
  T temp = std::move(t1); // or T temp(std::move(t1));
  t1 = std::move(t2);
  t2 = std::move(temp);
}
```

通过copy-and-swap的配合使用，可以有效地避免我们单独书写copy/move assignment operator时的两个问题：exception（必须保证如果`new`时出现了exception，`this`的值并没有改变） 和 self-assignment（手动判断`&other == this `的情况）

分别单独书写copy/move assignment operator的示例如下：

```c++
class MyClass {
  public:
    MyClass& operator=(const MyClass &other) {
      // 0. check for self-assignment
      if (&other != this) {
        // 1. allocate memory at local(maybe throw exception)
        int *new_arr = new int[other.size_];
        std::copy(other.arr_, other.arr_ + other.size_, new_arr);
        // 2. delete memory for this
        delete[] arr_;
        // 3. assign new value for this
        arr_ = new_arr;
      }
      return *this;
    }
    
    MyClass& operator=(MyClass &&other) {
      // 0. check for self-assignment
      if (&other != this) {
        // 1. delete memory for this
        delete[] arr_;
        // 2. assign Rvalue reference to this
        arr_ = other.arr_;
        // 3. let Rvalue reference in a destructible state
        other.arr_ = nullptr;
      }
      return *this;
    }
}
```

同样地，我们的move constructor也借助了copy-and-swap的方法（这里必须保证使用的default constructor不会造成任何内存资源申请，防止无意义的`new`操作）。如果不用copy-and-swap，单独书写move constructor的示例如下：

```c++
class MyClass {
  public:
    MyClass(MyClass &&other): size_(other.size_), arr_(other.arr_) {
      // important: let Rvalue reference in a destructible state
      other.arr_ = nullptr;
    }
}
```



## References

- [[CppReference] The rule of three/five/zero](https://en.cppreference.com/w/cpp/language/rule_of_three)
- [[Cpp Pattern] Copy-and-swap](https://cpppatterns.com/patterns/copy-and-swap.html)
- [[StackOverflow] What is the copy-and-swap idiom?](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom)
- [[StackOverflow] How does the standard library implement std::swap?](https://stackoverflow.com/questions/25286544/how-does-the-standard-library-implement-stdswap)
- [[StackOverflow] public friend swap member function](https://stackoverflow.com/questions/5695548/public-friend-swap-member-function)
- [[StackOverflow] What is "Argument-Dependent Lookup" (aka ADL, or "Koenig Lookup")?](https://stackoverflow.com/questions/8111677/what-is-argument-dependent-lookup-aka-adl-or-koenig-lookup)
- [[StackOverflow] How does "using std::swap" enable ADL?](https://stackoverflow.com/questions/28130671/how-does-using-stdswap-enable-adl)


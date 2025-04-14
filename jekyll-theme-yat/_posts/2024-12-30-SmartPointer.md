---
layout: post
title: "Pimpl"
categories: misc
---

- 议题-关于make_shared和make_unique

tips: std::make_shared是C++11的一部分，std::make_unique是C++14加入标准库的。

```C++
template<typename T,Typename... Ts> 
std::unique_ptr<T> make_unique(Ts&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```
make_unique是将形参向待创建对象的构造函数做了一次完美转发，从一个new运算符产生的裸指针出发，构造了一个std::unique_ptr，而后返回了如此创建的std::unique_ptr，这个形式的函数不支持数组和自定义析构器。但是证明了只需要一点点的努力就可以创建一个make_unique。

- 议题-make系列


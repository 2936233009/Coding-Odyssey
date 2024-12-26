---
layout: post
title: "Pimpl"
categories: misc
---

<link rel="stylesheet" href="jekyll-theme-yat/assets/css/style.css">

* 议题-关于减少编译依赖性

```C++
class Widget{
    public:
        Widget();
        ...
    private:
        std::string name;
        std::vector<double> data;
        Gadget g1, g2, g3;            // Gadget 是某种用户自定义型别
}
```

事实上，Widget的数据成员属于std::string, std::vector还有Gadget等多种型别，这些型别所对应的头文件必须存在，，widget才能通过编译，这就说明Widget的客户必须include<string>, include<vector>, 以及gadget.h。这些头文件增加了了Widget的客户的编译时间，此外，也使得这些客户依赖于这些头文件的内容。


* 具体内容

```C++
// widget.h
class Widget{
    public:
        Widget();
        ~Widget();   //仅声明
        ...
    private:
        struct Impl;
        std::unique<Impl> pImpl; //使用智能指针而非裸指针
}

// widget.cpp
#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl{
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;
};

Widget::Widget() :pImpl(std::make_unique<Impl>()){} 
Widget::~Widget() {} // ~Widget的定义
```

```C++
// 或者直接在 Widget.h中声明
// 表示
Widget::~Widget() = default;
```
tpis: 在 C++ 中，构造函数后面加上 = default 的语法，表示显式地要求编译器生成该构造函数的默认实现。

* 议题=关于移动操作

假设需要支持移动操作，在Widget中声明析构函数的举动会阻止编译器产生移动操作，所以假设需要支持移动操作，就必须自己声明该操作；

直接复制以上操作的话：

```C++
// widget.h
Widget(Widget&& rhs) = default;              //想法正确，产生代码错误
Widget& operator = (Widget&& rhs) = default; //
```

这种手法会导致和类中没有声明析构函数一样的问题，产生该问题的基本原因也相同，编译器生成的移动赋值操作需要在重新赋值之前析构pImpl指针涉及到的内容，但在Widget的头文件里pImpl指针涉及到的是非完整型别，move构造函数出问题的原因有所不同，这里的问题在于，编译器会在move的构造函数内抛出异常的事件中生成析构pImpl的代码，而对pImpl析构要求pImpl具备完整型别。

```C++
//Widget.h
...
Widget(Widget&& rhs);            //仅仅声明
Widget& operator=(Widget&& rhs);
...

//Widget.cpp
Widget::Widget(Widget&& rhs) =default;             //在这里放置定义
Widget& Widget::operator= (Widget&& rhs) = default;// 
```

* 议题-关于深复制

如果Gadget类像vector、string一样可以复制，那么Widget也可以支持复制操作；
由于 
1. 编译器不会像std::unique_ptr那样的只移型别生成复制操作，；
2. 即使编译器可以生成，其生成的函数也只能复制std::unique_ptr（即，实施的是浅复制），而我们希望的是复制指针所涉及到的内容

实现如下：

```C++
//Widget.h
Widget();
~Widget();

//Widget.cpp
Widget::Widget(const Widget& rhs):pImpl(std::make_unique<Impl>(*rhs.pImpl)){} //复制构造函数
Widget& Wudget::operator=(const Widget&rhs){                                  //复制赋值运算符
    *pImpl =*rhs.pImpl;
    return *this;
}
```
* 议题-关于make_unique和make_shared

对于std::make_unique而言，析构器型别是智能指针型别的一部分，这使得编译器会产生更小尺寸的运行期数据结构以及更快速的运行期代码，如此搞笑带来的后果就是，想要使用编译器生成的特种函数们就要求其指涉及到的型别必须是完整型别，而对于std::shared_ptr而言，析构器的型别并非是智能指针的一部分，这就需要更大尺寸的运行时期数据结构以及更慢一些的目标代码，但在使用编译器生成的特种函数时，其指涉及到型别并不要求是完整型别。


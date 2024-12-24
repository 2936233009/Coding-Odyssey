---
layout: post
title: "Pimpl"
categories: misc
---

<link rel="stylesheet" href="jekyll-theme-yat/assets/css/style.css">

* 引言

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

* 引言

假设需要支持移动操作：


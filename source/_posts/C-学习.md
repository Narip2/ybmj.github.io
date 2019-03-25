---
title: C++学习
date: 2019-03-20 20:28:47
categories:
- C++
tags:
- C++
---

### const *p 与 * const p 的区别
`int const *p` 表示`*p`是常量，即不能修改`*p`的内容，但是可以修改`p`的指向。
`int * const p` 表示`p`是常量，即不能修改`p`的指向，但是可以修改`*p`的内容。

### volatile
`volatile int i;` 表示`i`是易变的，不允许编译器对其优化，每次用i都要从其地址中读取。
```cpp
volatile int i = 10;
int j = i, k = i;
```
对上述代码，编译器会优化为：在`j`赋值之后，会将此时的`i`值保存下来，从而在对`k`赋值的时候，就不需要重新访问`i`的地址了。
但如果`i`声明为易变的变量，就表示`i`的值随时可能发生变化（比如寄存器变量，或某个端口的数据），在两次调用期间可能`i`值会发生改变。

### C++内存空间划分
- 代码段：用来存放程序的执行代码。
- 数据段：
    - BSS段：用来存放程序中未初始化的外部变量和未初始化的静态局部变量。
    - 静态数据区：存放已初始化的外部变量，静态局部变量和常量。
- 堆空间：存放程序执行中动态分配的内存段。存放全局对象。
- 栈空间：存放局部变量，函数参数和函数返回地址等。

### new与malloc的区别
new是操作符，分配内存后会调用构造函数。
malloc是库函数，单纯分配一块内存，需要进行类型转换。

### mutable
`mutable int i;` 表示`i`是永远可变的，即使`i`是某个`const`对象的数据成员。

### 友元关系的特点
- 友元关系是非传递的，例如类B是类A的友元类，类C是类B的友元类，在类C和类A之间并无友元关系。
- 友元关系是单向的，例如类B是类A的友元类，但是类A的成员函数不能访问类B的私有和保护数据。
- 友元关系不能继承。

### 类模板和模板类
![](https://ybmj-blog-1256173108.cos.ap-shanghai.myqcloud.com/blog-picture/FAC1A1AE-30FF-4058-95F0-B7CBED1B4653.jpeg?q-sign-algorithm=sha1&q-ak=AKIDMdujyB3ilczYc2qEljxvrDUUfV9jzJbv&q-sign-time=1553097040;1553098840&q-key-time=1553097040;1553098840&q-header-list=&q-url-param-list=&q-signature=65d644a245c3b1faaadb8487d2f1d72370e1d534&x-cos-security-token=09ec0b6cac6af646b33b5bf4d04b52848289a46610001)

函数模板与类模板的区别：
函数模板的实例化是由编译程序在处理函数调用时自动完成的，而类模板的实例化必须由程序员在程序中显式地指定。
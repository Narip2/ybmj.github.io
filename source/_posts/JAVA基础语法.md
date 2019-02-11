---
title: JAVA基础
date: 2019-02-11 20:23:08
categories:
- JAVA
tags:
- JAVA
---


### 第一个JAVA程序

```java
public class FirstSample{
    public static void main(String[] args){
        System.out.println("Fuk the world!");
    }
}
```
{% label success@public: %} 访问修饰符

#### 访问修饰符

{% tabs access modifier%}
<!-- tab Public-->
**对于类：** 类外可用
**对于方法：** 类外可用
**对于变量：** 类外可用
<!-- endtab -->

<!-- tab Private-->
**对于类：** 类内可用. 一般用于*内部类*，即这个类创建在一个类的内部，且不希望被外部访问。
**对于方法：** 类内可用
**对于变量：** 类内可用. 建议所有的变量都设置为**private**
<!-- endtab -->

<!-- tab Protected-->
**对于类：** 类内和子类可用
**对于方法：** 类内和子类可用
**对于变量：** 类内和子类可用
<!-- endtab -->
{% endtabs %}

{% label info@class: %} 类定义的关键字。 JAVA中，一切都是对象，所有的代码都必须在某个类中。

{% note danger %}
#### 源代码的文件名必须与公共类的名字相同，并用.java作为扩展名
因此上述代码的文件名必须为FirstSample.java
{% endnote %}

运行命令`javac FirstSample`进行编译，编译后会生成相应的.class文件
这时候运行命令`java FirstSample`即可运行

{% label primary@main：%} 每个类都可以有一个$main$函数，当我们按上述方式运行时，JAVA虚拟机将从指定类中的$main$方法开始执行。

#### Static
静态变量：类的多个实体之间共同拥有的属性。
静态方法：修改静态变量的唯一途径，不需要通过实体即可调用。

非静态的方法，都需要通过实体来调用
```java
System.out.println("Fuk the world!");
```
$System$是一个类，$out$是一个实体，$println$是类的方法

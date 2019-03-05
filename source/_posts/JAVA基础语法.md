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
<!--more-->

**{% label success@public: %}** 访问修饰符

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

**{% label info@class: %}** 类定义的关键字。 JAVA中，一切都是对象，所有的代码都必须在某个类中。

{% note danger %}
#### 源代码的文件名必须与公共类的名字相同，并用.java作为扩展名
因此上述代码的文件名必须为FirstSample.java
{% endnote %}

运行命令`javac FirstSample`进行编译，编译后会生成相应的.class文件
这时候运行命令`java FirstSample`即可运行

**{% label primary@main：%}** 每个类都可以有一个$main$函数，当我们按上述方式运行时，JAVA虚拟机将从指定类中的$main$方法开始执行。

#### Static
静态变量：类的多个实体之间共同拥有的属性。
静态方法：修改静态变量的唯一途径，不需要通过实体即可调用。

非静态的方法，都需要通过实体来调用
```java
System.out.println("Fuk the world!");
```
$System$是一个类，$out$是一个实体，$println$是类的方法

---

### 注释：自动生成文档
以/\*\*开始，以\*/结束

```java
/**
 * @author 标识一个类的作者
 * @version 指定类的版本
 * @param 方法的参数
 * @return 返回值描述
 * @exception 可能抛出的异常
 * @throws 同上 
 * @see 参考
 * @since 用于标识编译该文件所需要的JDK环境
 * @deprecated 指名一个过期的类，成员或方法
 */
```

### 数据类型

在JAVA中，一共有8种数据类型，除此之外全是对象。

|类型|字节数|备注|
|-----|-----|-----|
|int|4| |
|short|2| |
|long|8| |
|byte|1| |
|float|4|有效位数在6～7位。通常以后缀加F或f表示|
|double|8|有效位数15位。浮点数默认类型|
|char|2|Unicode。 避免使用char。 小心\u,即使是注释中的|
|boolean| |单独使用占4个字节，数组的话是1个字节|

#### 三种特殊的浮点数值
正无穷大：正整数除以0的结果
负无穷大：正无穷大取负
NaN：0/0或负数的平方根

{% note success %}
常量`Double.POSITIVE_INFINITY`,`Double.NEGATIVE_INFINITY`和`Double.NaN`来分别表示这三个特殊的值
{% endnote %}

**判断方法：**
```java
if(x == Double.NaN) // not ok! is never true.
if(Double.isNaN(x)) // right!
```

**boolean类型**: 整数值和布尔值之间不能进行相互转化。

**常量**： 用$final$ 表示常量，只能被赋值一次。

### 字符串

$String$ 属于不可变字符串。

**判断字符串相等：**
```java
s.equals(t);
```

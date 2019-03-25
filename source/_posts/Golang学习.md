---
title: Golang学习
date: 2019-03-22 22:42:01
categories:
- Golang
tags:
- Golang
---
### Blank Identifier
```
Go defines a special identifier _, called the blank identifier. 
The blank identifier can be used in a declaration to avoid declaring a name, 
and it can be used in an assignment to discard a value. 
This definition makes it useful in a variety of contexts.
```

[参考博客](https://studygolang.com/articles/425)

### 第三方包下载
到`https://github.com/golang?page=1` 找到要下载的包
然后克隆到`GOPATH/src/golang.org/x/`下即可

### 多变量赋值
```go
x,y = y,x
```
在赋值操作之前会先对等号右边所有表达式求值，然后再同一更新左边的变量。
这也是为什么上面的代码可以这么写。

### Golang中的自动垃圾回收机制
Garbage Collection(GC)的大致工作原理：
Go采用了一种标记-清扫的办法来进行垃圾回收。程序运行一段时间或垃圾达到一定阈值后，系统会运行STW(stop the world)程序，使所有程序休眠。
然后gc会从所有的包变量和当前运行函数的局部变量开始搜索，查找所有与之关联的变量。对于那些不可达的变量会进行内存回收。

对程序编写的直观改变就是对于申请内存，无需手动delete。 但是gc也无疑会增加程序的运行时间，这个留到以后遇到问题再进行深入研究。

### 数组的传递
在Golang中，数组作为参数传递时，函数接收到的是一个副本，并不是原数组的引用。
所以如果想要在函数中直接修改数组，则需要传入数组的指针。

### for range
```
The range expression x is evaluated once before beginning the loop, with one exception: 
if at most one iteration variable is present and len(x) is constant, the range expression is not evaluated.
```

[参考博客](https://studygolang.com/articles/9703)

```go
arr := []int{1, 2, 3}
var a []int
var b []*int
for _, i := range arr {
    a = append(a, i)
    b = append(b, &i)
}
for _, i := range a {
    fmt.Println(i)
}
for _, i := range b {
    fmt.Println(i)
}
```
输出：
```
1
2
3
0xc00005a058
0xc00005a058
0xc00005a058
```
我们发现循环变量$i$的地址是不变的。
实际上的循环实现大致如下：
```
Arrange to do a loop appropriate for the type.  We will produce
for INIT ; COND ; POST {
        ITER_INIT
        INDEX = INDEX_TEMP
        VALUE = VALUE_TEMP // If there is a value
        original statements
}
```
`VALUE`是重复使用的一个临时变量，它的地址是不会改变的。
这也解释了为什么输出的地址都是一样的。

### Defer
```
A "defer" statement invokes a function whose execution is deferred to the 
moment the surrounding function returns, 
either because the surrounding function executed a return statement, 
reached the end of its function body, or because the corresponding goroutine is panicking.
```
被延迟的函数，它的执行会在它的`surrounding function`返回之前执行。
具体来说
```go
func main() {
    fmt.Println(f())
}
func f() (result int) {
    defer func() {
        result++
    }()
    return 0
}
```
输出：
```
1
```

匿名函数的执行会在`f()`返回时执行。
`f()`的返回分为两步：
- result = 0
- return result

而被延迟的函数就会在第一步执行完后开始执行。因此输出为`1`

如果`surrounding function`中有多个`defer`的函数，那么它们的执行顺序满足先进后出。

Defer可以用于标识那些`surrounding function`结束之前**必须要执行的操作**

[参考文献](https://blog.csdn.net/wangshubo1989/article/details/74357138)
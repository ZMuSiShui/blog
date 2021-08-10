---
title: Golang 学习笔记(三)
date: 2021-08-10 17:21:42
tags:
  - Golang
  - 学习笔记
categories:
  - Golang
---

# 流程控制

Go中流程控制分三大类：条件判断，循环控制和无条件跳转。

## if

`if`也许是各种编程语言中最常见的了，它的语法概括起来就是:如果满足条件就做某事，否则做另一件事。

Go里面`if`条件判断语句中不需要括号，如下代码所示

```go
if x > 60 {
    fmt.Println("及格了")
} else {
    fmt.Println("不及格")
}
```

Go的`if`条件判断语句里面允许声明变量，这个变量的作用域只能在该条件逻辑块内
如下所示:

```go
// 计算获取值x,然后根据x返回的大小，判断是否大于10。
if x := computedValue(); x > 10 {
    fmt.Println("x 大于 10")
} else {
    fmt.Println("x 小于 10")
}

//这个地方如果这样调用就编译出错了，因为x是if条件块里面的变量
fmt.Println(x)
```

多个条件的时候如下所示：

```go
if integer == 3 {
    fmt.Println("等于3")
} else if integer < 3 {
    fmt.Println("小于3")
} else {
    fmt.Println("大于3")
}
```

## goto

用`goto`跳转到必须在当前函数内定义的标签。例如假设这样一个循环：

```go
func myFunc() {
    i := 0
Here:   //这行的第一个词，以冒号结束作为标签
    println(i)
    i++
    goto Here   //跳转到Here去
}
```

标签名是大小写敏感的。

## for

Go里面最强大的一个控制逻辑就是`for`，它即可以用来循环读取数据，又可以当作`while`来控制逻辑，还能迭代操作。

语法如下：

```go
for expression1; expression2; expression3 {
    //...
}
```

`expression1`、`expression2`和`expression3`都是表达式， 其中`expression1`和`expression3`是变量声明或者函数调用返回值之类的， `expression2`是用来条件判断，`expression1`在循环开始之前调用，`expression3`在每轮循环结束之时调用。

示例：

```go
package main
import "fmt"

func main(){
    sum := 0;
    for index := 0; index < 10; index++ {
        sum += index
    }
    fmt.Println("sum is equal to ", sum)
}
```

在循环里面有两个关键操作`break`和`continue` ,`break`操作是跳出当前循环，`continue`是跳过本次循环。 当嵌套过深的时候，`break`可以配合标签使用，即跳转至标签所指定的位置

```go
for index := 10; index > 0; index-- {
    if index == 5 {
        break // 或者continue
    }
    fmt.Println(index)
}
```

`break`和`continue`还可以跟着标签，用来跳到多重循环中的外层循环

`for`配合`range`可以用于读取`slice`和`map`的数据：

```go
for k,v := range mapValue {
    fmt.Println("map's key:",k)
    fmt.Println("map's val:",v)
}
```

由于 Go 支持 “多值返回”, 而对于“声明而未被调用”的变量, 编译器会报错, 在这种情况下, 可以使用`_`来丢弃不需要的返回值 例如

```go
for _, v := range mapValue {
    fmt.Println("map's val:", v)
}
```

## switch

有时候需要写很多的`if-else`来实现一些逻辑处理，这时代码看上去就很丑很冗长，而且也不易于以后的维护， 这时`switch`就能很好的解决这个问题

```go
switch sExpr {
case expr1:
    some instructions
case expr2:
    some other instructions
case expr3:
    some other instructions
default:
    other code
}
```

`sExpr`和`expr1`、`expr2`、`expr3`的类型必须一致。 Go的`switch`非常灵活，表达式不必是常量或整数，执行的过程从上至下，直到找到匹配项； 而如果`switch`没有表达式，它会匹配`true`。

```go
i := 10
switch i {
case 1:
    fmt.Println("i is equal to 1")
case 2, 3, 4:
    fmt.Println("i is equal to 2, 3 or 4")
case 10:
    fmt.Println("i is equal to 10")
default:
    fmt.Println("All I know is that i is an integer")
}

// 无判断条件，选择结果为true的那个
switch {
case i > 0:
    fmt.Println("i is larger than 0")
case i = 0:
    fmt.Println("i is 0")
default:
    fmt.Println("i is smaller than 0")
}
```

Go里面`switch`默认每个`case`最后带有`break`，匹配成功后不会自动向下执行其他case，而是跳出整个`switch`。 但是可以使用`fallthrough`强制执行后面的case代码。

```go
integer := 6
switch integer {
    case 4:
        fmt.Println("The integer was <= 4")
        fallthrough
    case 5:
        fmt.Println("The integer was <= 5")
        fallthrough
    case 6:
        fmt.Println("The integer was <= 6")
        fallthrough
    case 7:
        fmt.Println("The integer was <= 7")
        fallthrough
    case 8:
        fmt.Println("The integer was <= 8")
        fallthrough
    default:
        fmt.Println("default case")
}
```

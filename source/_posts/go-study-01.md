---
title: Golang 学习笔记
date: 2021-08-09 21:46:46
tags:
  - Golang
  - 学习笔记
categories:
  - Golang
---

# 环境安装

本机系统：Windows 11 专业版 21H2

Go 版本：1.16.5 windows/amd64

- 下载安装包

  地址：https://golang.org/dl/go1.16.5.windows-amd64.msi

- 安装完成后检查

  ```shell
  PS D:\Project\godemo> go version
  go version go1.16.5 windows/amd64
  ```

- 检查 Go env

  ```shell
  PS D:\Project\godemo> go env
  set GO111MODULE=on
  set GOARCH=amd64
  set GOBIN=
  set GOCACHE=C:\Users\zhang\AppData\Local\go-build
  set GOENV=C:\Users\zhang\AppData\Roaming\go\env
  set GOEXE=.exe
  set GOFLAGS=
  set GOHOSTARCH=amd64
  set GOHOSTOS=windows
  set GOINSECURE=
  set GOMODCACHE=C:\Users\zhang\go\pkg\mod
  set GONOPROXY=
  set GONOSUMDB=
  set GOOS=windows
  set GOPATH=C:\Users\zhang\go
  set GOPRIVATE=
  set GOPROXY=https://goproxy.cn,direct
  set GOROOT=D:\Program Files\Go
  set GOSUMDB=sum.golang.org
  set GOTMPDIR=
  set GOTOOLDIR=D:\Program Files\Go\pkg\tool\windows_amd64
  set GOVCS=
  set GOVERSION=go1.16.5
  set GCCGO=gccgo
  set AR=ar
  set CC=gcc
  set CXX=g++
  set CGO_ENABLED=1
  set GOMOD=NUL
  set CGO_CFLAGS=-g -O2
  set CGO_CPPFLAGS=
  set CGO_CXXFLAGS=-g -O2
  set CGO_FFLAGS=-g -O2
  set CGO_LDFLAGS=-g -O2
  set PKG_CONFIG=pkg-config
  set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\zhang\AppData\Local\Temp\go-build3804193883=/tmp/go-build -gno-record-gcc-switches
  ```

- Hello World

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	fmt.Println("Hello World!")
  }
  ```

  命令行执行：

  ```
  PS D:\Project\godemo> go run main.go
  Hello World!
  ```

## Go 命令

```
PS D:\Project\godemo> go
Go is a tool for managing Go source code.

Usage:

        go <command> [arguments]

The commands are:

        bug         start a bug report
        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         add dependencies to current module and install them
        install     compile and install packages and dependencies
        list        list packages or modules
        mod         module maintenance
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages
```

- go run main

  不生成任何文件只运行程序。

- go build main

  在对应当前目录下编译生成文件。

- go install main

  会把编译好的结果移动到 $GOPATH/bin。

- go fmt main

  格式化代码，将代码修改成标准格式。

## 开发工具

VS code

![Visual Studio Code - 维基百科，自由的百科全书](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Visual_Studio_Code_1.35_icon.svg/1200px-Visual_Studio_Code_1.35_icon.svg.png)

# 常量与变量

## 数据类型

- 空值：nil

- 整型： 
  - int8, int16, int32, int64
  - uint8, uint16, uint32, uint64
  - int, uint, 具体长度取决于 CPU 位数

- 浮点数：
  - float32
  - float64

- 字节：byte (等价于uint8)

- 字符串：string

  只能用一对双引号（""）或反引号（``）括起来定义，不能用单引号（''）定义！

- 布尔值：boolean

  只有 true 和 false，默认为 false。

## 常量声明

**常量**，在程序编译阶段就确定下来的值，而程序在运行时无法改变该值。

### 单个常量声明

- const 变量名称 数据类型 = 变量值

  如果不赋值，使用的是该数据类型的默认值。

- const 变量名称 = 变量值

  根据变量值，自行判断数据类型。

### 多个常量声明

- const 变量名称,变量名称 ... ,数据类型 = 变量值,变量值 ...
- const 变量名称,变量名称 ... = 变量值,变量值 ...

```go
// constant.go
package main

import "fmt"

func main() {
	const name string = "张三"
	fmt.Println(name)

	const age int = 25
	fmt.Println(age)

	const name_1, name_2 string = "王二", "麻子"
	fmt.Println(name_1, name_2)

	const name_3, age_3 = "李四", 24
	fmt.Println(name_3, age_3)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run constant.go
张三
25
王二 麻子
李四 24
```

## 变量声明

### 单个变量声明

- var 变量名称 数据类型 = 变量值

  如果不赋值，使用的是该数据类型的默认值。

- var 变量名称 = 变量值

  根据变量值，自行判断数据类型。

- 变量名称 := 变量值

  省略了 var 和数据类型，变量名称一定要是未声明过的。

### 多个变量声明

- var 变量名称,变量名称 ... ,数据类型 = 变量值,变量值 ...
- var 变量名称,变量名称 ... = 变量值,变量值 ...
- 变量名称,变量名称 ... := 变量值,变量值 ...

```go
// variable.go
package main

import (
	"fmt"
)

func main() {
	var age_1 uint8 = 31
	var age_2 = 32
	age_3 := 33
	fmt.Println(age_1, age_2, age_3)

	var age_4, age_5, age_6 int = 31, 32, 33
	fmt.Println(age_4, age_5, age_6)

	var name_1, age_7 = "Tom", 30
	fmt.Println(name_1, age_7)

	name_2, is_boy, height := "Jay", true, 180.66
	fmt.Println(name_2, is_boy, height)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run variable.go
31 32 33
31 32 33
Tom 30
Jay true 180.66
```

## 输出方法

- **fmt.Print**

  输出到控制台（不换行）

- **fmt.Println**

  输出到控制台并换行

- **fmt.Printf**

  仅输出格式化的字符串和字符串变量（整型和整型变量不可以）

- **fmt.Sprintf**

  格式化并返回一个字符串，不输出

```go
// output.go
package main

import (
	"fmt"
)

func main() {
	fmt.Print("输出到控制台不换行")
	fmt.Println("---")
	fmt.Println("输出到控制台并换行")
	fmt.Printf("name=%s,age=%d\n", "Tom", 30)
	fmt.Printf("name=%s,age=%d,height=%v\n", "Tom", 30, fmt.Sprintf("%.2f", 180.567))
}
```

运行结果：

```shell
PS D:\Project\godemo> go run output.go
输出到控制台不换行---
输出到控制台并换行
name=Tom,age=30
name=Tom,age=30,height=180.57
```


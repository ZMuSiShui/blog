---
title: Golang 学习笔记(四)
date: 2021-08-10 21:53:56
tags:
  - Golang
  - 学习笔记
categories:
  - Golang
---

# 函数

## 函数定义

```go
func function_name(input1 type1, input2 type2) (type1, type2) {
   // 函数体
   // 返回多个值
   return value1, value2
}
```

- 函数用 `func` 声明。
- 函数可以有一个或多个参数，需要有参数类型，用 `,` 分割。
- 函数可以有一个或多个返回值，需要有返回值类型，用 `,` 分割。
- 函数的参数是可选的，返回值也是可选的。

## 值传递

传递参数时，将参数复制一份传递到函数中，对参数进行调整后，不影响参数值。

Go 语言默认是值传递。

## 引用传递

传递参数时，将参数的地址传递到函数中，对参数进行调整后，影响参数值。

```go
// pointer.go
package main

import (
    "encoding/json"
    "fmt"
)

type Result struct {
    Code    int    `json:"code"`
    Message string `json:"msg"`
}

func main() {
    var res Result
    res.Code    = 200
    res.Message = "success"
    toJson(&res)
    
    setData(&res)
    toJson(&res)
}

func setData (res *Result) {
    res.Code    = 500
    res.Message = "fail"
}

func toJson (res *Result) {
    jsons, errs := json.Marshal(res)
    if errs != nil {
        fmt.Println("json marshal error:", errs)
    }
    fmt.Println("json data :", string(jsons))
}
```

运行结果：

```shell
PS D:\Project\godemo> go run pointer.go
json data : {"code":200,"msg":"success"}
json data : {"code":500,"msg":"fail"}
```

## 返回值

示例，实现2个数的加法（一个返回值）和除法（多个返回值）：

```go
// return_demo.go
package main

import "fmt"

func add(num1 int, num2 int) int {
	return num1 + num2
}

func div(num1 int, num2 int) (int, int) {
	return num1 / num2, num1 % num2
}
func main() {
	quo, rem := div(100, 17)
	fmt.Println(quo, rem)
	fmt.Println(add(100, 17))
}
```

运行结果：

```shell
PS D:\Project\godemo> go run return_demo.go
5 15
117
```

也可以给返回值命名，简化 return，例如 add 函数可以改写为

```go
func add(num1 int, num2 int) (ans int) {
	ans = num1 + num2
	return
}
```

## 错误处理(error handling)

如果函数实现过程中，如果出现不能处理的错误，可以返回给调用者处理。比如我们调用标准库函数`os.Open`读取文件，`os.Open` 有2个返回值，第一个是 `*File`，第二个是 `error`， 如果调用成功，error 的值是 nil，如果调用失败，例如文件不存在，我们可以通过 error 知道具体的错误信息。

```go
// error_demo.go
package main

import (
	"fmt"
	"os"
)

func main() {
	_, err := os.Open("filename.txt")
	if err != nil {
		fmt.Println(err)
	}
}
```

文件不存在时，运行结果：

```shell
PS D:\Project\godemo> go run error_demo.go
open filename.txt: The system cannot find the file specified.
```

### 可以通过 `errorw.New` 返回自定义的错误

```go
// error_demo1.go
package main

import (
	"errors"
	"fmt"
)

func hello(name string) error {
	if len(name) == 0 {
		return errors.New("error: name is null")
	}
	fmt.Println("Hello,", name)
	return nil
}

func main() {
	if err := hello(""); err != nil {
		fmt.Println(err)
	}
}
```

运行结果：

```shell
PS D:\Project\godemo> go run error_demo1.go
error: name is null
```

**error 往往是能预知的错误，但是也可能出现一些不可预知的错误，例如数组越界，这种错误可能会导致程序非正常退出，在 Go 语言中称之为 panic。**

```go
// error_demo2.go
package main

import "fmt"

func get(index int) int {
	arr := [3]int{2, 3, 4}
	return arr[index]
}

func main() {
	fmt.Println(get(5))
	fmt.Println("finished")
}
```

运行结果：

```shell
PS D:\Project\godemo> go run error_demo2.go 
panic: runtime error: index out of range [5] with length 3

goroutine 1 [running]:
main.get(...)
        D:/Project/godemo/error_demo.go:7
main.main()
        D:/Project/godemo/error_demo.go:11 +0x1d
exit status 2
```

- 在 get 函数中，使用 defer 定义了异常处理的函数，在协程退出前，会执行完 defer 挂载的任务。因此如果触发了 panic，控制权就交给了 defer。
- 在 defer 的处理逻辑中，使用 recover，使程序恢复正常，并且将返回值设置为 -1，在这里也可以不处理返回值，如果不处理返回值，返回值将被置为默认值 0。

## 计算 MD5

```go
// MD5
package main

import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
)

func MD5(str string) string {
	s := md5.New()
	s.Write([]byte(str))
	return hex.EncodeToString(s.Sum(nil))
}

func main() {
	str := "12345"
	fmt.Printf("MD5(%s): %s\n", str, MD5(str))
}s
```

运行结果：

```shell
PS D:\Project\godemo> go run md5_demo.go
MD5(12345): 827ccb0eea8a706c4c34a16891f84e7b
```

## 获取时间戳

```go
// timestamp.go
package main

import (
	"fmt"
	"time"
)

func getTimeInt() int64 {
	return time.Now().Unix()
}
func getTimeStr() string {
	return time.Now().Format("2006-01-02 15:04:05")
}

func main() {
	fmt.Printf("current time str : %s\n", getTimeStr())
	fmt.Printf("current time unix : %d\n", getTimeInt())
}
```

运行结果：

```shell
PS D:\Project\godemo> go run timestamp.go
current time str : 2021-08-10 22:14:40
current time unix : 1628604880
```

## 生成签名

```go
// sign.go
package main

import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
	"sort"
)

func main() {
	params := map[string]interface{} {
		"name" : "Tom",
		"pwd"  : "123456",
		"age"  : 30,
	}
	fmt.Printf("sign : %s\n", createSign(params))
}

// MD5 方法
func MD5(str string) string {
	s := md5.New()
	s.Write([]byte(str))
	return hex.EncodeToString(s.Sum(nil))
}

// 生成签名
func createSign(params map[string]interface{}) string {
	var key []string
	var str = ""
	for k := range params {
		key   = append(key, k)
	}
	sort.Strings(key)
	for i := 0; i < len(key); i++ {
		if i == 0 {
			str = fmt.Sprintf("%v=%v", key[i], params[key[i]])
		} else {
			str = str + fmt.Sprintf("&xl_%v=%v", key[i], params[key[i]])
		}
	}
	// 自定义密钥
	var secret = "123456789"

	// 自定义签名算法
	sign := MD5(MD5(str) + MD5(secret))
	return sign
}
```

运行结果：

```shell
PS D:\Project\godemo> go run sign.go
sign : 33b940c8f18ede393ea34cd45c406db4
```


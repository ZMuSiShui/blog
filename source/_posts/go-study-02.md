---
title: Golang 学习笔记(二)
date: 2021-08-10 14:19:20
tags:
  - Golang
  - 学习笔记
categories:
  - Golang
---

# 数组与切片

## 数组(array)

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成，一旦声明了，数组的长度就固定了，不能动态变化。

`len()` 和 `cap()` 返回结果始终一样。

### 声明数组

```go
// declareArr.go
package main

import (
	"fmt"
)

func main() {
	//一维数组
	var arr_1 [5] int
	fmt.Println(arr_1)

	var arr_2 =  [5] int {1, 2, 3, 4, 5}
	fmt.Println(arr_2)

	arr_3 := [5] int {1, 2, 3, 4, 5}
	fmt.Println(arr_3)

	arr_4 := [...] int {1, 2, 3, 4, 5, 6}
	fmt.Println(arr_4)

	arr_5 := [5] int {0:3, 1:5, 4:6}
	fmt.Println(arr_5)

	//二维数组
	var arr_6 = [3][5] int {{1, 2, 3, 4, 5}, {9, 8, 7, 6, 5}, {3, 4, 5, 6, 7}}
	fmt.Println(arr_6)

	arr_7 :=  [3][5] int {{1, 2, 3, 4, 5}, {9, 8, 7, 6, 5}, {3, 4, 5, 6, 7}}
	fmt.Println(arr_7)

	arr_8 :=  [...][5] int {{1, 2, 3, 4, 5}, {9, 8, 7, 6, 5}, {0:3, 1:5, 4:6}}
	fmt.Println(arr_8)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run declareArr.go
[0 0 0 0 0]
[1 2 3 4 5]
[1 2 3 4 5]
[1 2 3 4 5 6]
[3 5 0 0 6]
[[1 2 3 4 5] [9 8 7 6 5] [3 4 5 6 7]]
[[1 2 3 4 5] [9 8 7 6 5] [3 4 5 6 7]]
[[1 2 3 4 5] [9 8 7 6 5] [3 5 0 0 6]]
```

### 注意事项

1. 数组不可动态变化问题，一旦声明了，其长度就是固定的。

   ```go
   var arr_1 = [5] int {1, 2, 3, 4, 5}
   arr_1[5] = 6
   fmt.Println(arr_1)
   ```

   运行会报错：invalid array index 5 (out of bounds for 5-element array)

2. 数组是值类型问题，在函数中传递的时候是传递的值，如果传递数组很大，这对内存是很大开销。

   ```go
   // arrDemo1.go
   package main
   
   import (
   	"fmt"
   )
   
   func main() {
   	var arr =  [5] int {1, 2, 3, 4, 5}
   	modifyArr(arr)
   	fmt.Println(arr)
   }
   
   func modifyArr(a [5] int) {
   	a[1] = 20
   }
   ```

   运行结果：

   ```shell
   PS D:\Project\godemo> go run arrDemo1.go
   [1 2 3 4 5]
   ```

   ```go
   // arrDemo2.go
   package main
   
   import (
   	"fmt"
   )
   
   func main() {
   	var arr =  [5] int {1, 2, 3, 4, 5}
   	modifyArr(&arr)
   	fmt.Println(arr)
   }
   
   func modifyArr(a *[5] int) {
   	a[1] = 20
   }
   ```

   运行结果：

   ```shell
   PS D:\Project\godemo> go run arrDemo2.go
   [1 20 3 4 5]
   ```

3. 数组赋值问题，同样类型的数组（长度一样且每个元素类型也一样）才可以相互赋值，反之不可以。

   ```go
   var arr =  [5] int {1, 2, 3, 4, 5}
   var arr_1 [5] int = arr
   var arr_2 [6] int = arr
   ```

   运行会报错：cannot use arr (type [5]int) as type [6]int in assignment

## 切片(slice)

切片是一种动态数组，比数组操作灵活，长度不是固定的，可以进行追加和删除。

`len()` 和 `cap()` 返回结果可相同和不同。

### 声明切片

```go
// declareSlice.go
package main

import (
	"fmt"
)

func main() {
	var sli_1 [] int      //nil 切片
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_1),cap(sli_1),sli_1)

	var sli_2 = [] int {} //空切片
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_1),cap(sli_2),sli_2)

	var sli_3 = [] int {1, 2, 3, 4, 5}
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_3),cap(sli_3),sli_3)

	sli_4 := [] int {1, 2, 3, 4, 5}
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_4),cap(sli_4),sli_4)

	var sli_5 [] int = make([] int, 5, 8)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_5),cap(sli_5),sli_5)

	sli_6 := make([] int, 5, 9)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli_6),cap(sli_6),sli_6)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run declareSlice.go
len=0 cap=0 slice=[]
len=0 cap=0 slice=[]
len=5 cap=5 slice=[1 2 3 4 5]
len=5 cap=5 slice=[1 2 3 4 5]
len=5 cap=8 slice=[0 0 0 0 0]
len=5 cap=9 slice=[0 0 0 0 0]
```

### 截取切片

```go
// cutSlice.go
package main

import (
	"fmt"
)

func main() {
	sli := [] int {1, 2, 3, 4, 5, 6}
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	fmt.Println("sli[1] ==", sli[1])
	fmt.Println("sli[:] ==", sli[:])
	fmt.Println("sli[1:] ==", sli[1:])
	fmt.Println("sli[:4] ==", sli[:4])
	
	fmt.Println("sli[0:3] ==", sli[0:3])
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli[0:3]),cap(sli[0:3]),sli[0:3])

	fmt.Println("sli[0:3:4] ==", sli[0:3:4])
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli[0:3:4]),cap(sli[0:3:4]),sli[0:3:4])
}
```

运行结果：

```shell
PS D:\Project\godemo> go run cutSlice.go
len=6 cap=6 slice=[1 2 3 4 5 6]
sli[1] == 2
sli[:] == [1 2 3 4 5 6]
sli[1:] == [2 3 4 5 6]
sli[:4] == [1 2 3 4]
sli[0:3] == [1 2 3]
len=3 cap=6 slice=[1 2 3]
sli[0:3:4] == [1 2 3]
len=3 cap=4 slice=[1 2 3]
```

### 追加切片

```go
// addSlice.go
package main

import (
	"fmt"
)

func main() {
	sli := [] int {4, 5, 6}
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	sli = append(sli, 7)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	sli = append(sli, 8)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	sli = append(sli, 9)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	sli = append(sli, 10)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run addSlice.go
len=3 cap=3 slice=[4 5 6]
len=4 cap=6 slice=[4 5 6 7]
len=5 cap=6 slice=[4 5 6 7 8]
len=6 cap=6 slice=[4 5 6 7 8 9]
len=7 cap=12 slice=[4 5 6 7 8 9 10]
```

append 时，容量不够需要扩容时，cap 会翻倍。

### 删除切片

```go
// delSlice.go
package main

import (
	"fmt"
)

func main() {
	sli := [] int {1, 2, 3, 4, 5, 6, 7, 8}
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)

	//删除尾部 2 个元素
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli[:len(sli)-2]),cap(sli[:len(sli)-2]),sli[:len(sli)-2])

	//删除开头 2 个元素
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli[2:]),cap(sli[2:]),sli[2:])

	//删除中间 2 个元素
	sli = append(sli[:3], sli[3+2:]...)
	fmt.Printf("len=%d cap=%d slice=%v\n",len(sli),cap(sli),sli)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run delSlice.go
len=8 cap=8 slice=[1 2 3 4 5 6 7 8]
len=6 cap=8 slice=[1 2 3 4 5 6]
len=6 cap=6 slice=[3 4 5 6 7 8]
len=6 cap=8 slice=[1 2 3 6 7 8]
```

# **Struct** 结构体

结构体是将零个或多个任意类型的变量，组合在一起的聚合数据类型，也可以看做是数据的集合。

## 结构体的声明

```go
// structDemo.go
package main

import (
	"fmt"
)

type Person struct {
	Name string
	Age int
}

func main() {
	var p1 Person
	p1.Name = "Tom"
	p1.Age  = 30
	fmt.Println("p1 =", p1)

	var p2 = Person{Name:"Burke", Age:31}
	fmt.Println("p2 =", p2)

	p3 := Person{Name:"Aaron", Age:32}
	fmt.Println("p2 =", p3)
	
	//匿名结构体
	p4 := struct {
		Name string
		Age int
	} {Name:"匿名", Age:33}
	fmt.Println("p4 =", p4)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run structDemo.go
p1 = {Tom 30}
p2 = {Burke 31}
p2 = {Aaron 32}
p4 = {匿名 33}
```

## 生成 JSON

```go
// structToJson.go
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

	//序列化
	jsons, errs := json.Marshal(res)
	if errs != nil {
		fmt.Println("json marshal error:", errs)
	}
	fmt.Println("json data :", string(jsons))

	//反序列化
	var res2 Result
	errs = json.Unmarshal(jsons, &res2)
	if errs != nil {
		fmt.Println("json unmarshal error:", errs)
	}
	fmt.Println("res2 :", res2)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run structToJson.go
json data : {"code":200,"msg":"success"}
res2 : {200 success}
```

## 改变数据

```go
// changeStructData.go
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
PS D:\Project\godemo> go run changeStructData.go
json data : {"code":200,"msg":"success"}
json data : {"code":500,"msg":"fail"}
```

# **Map** 集合

Map 集合是无序的 key-value 数据结构。

Map 集合中的 key / value 可以是任意类型，但所有的 key 必须属于同一数据类型，所有的 value 必须属于同一数据类型，key 和 value 的数据类型可以不相同。

## Map 的声明

```go
// declareMap.go
package main

import (
	"fmt"
)

func main() {
	var p1 map[int]string
	p1 = make(map[int]string)
	p1[1] = "Tom"
	fmt.Println("p1 :", p1)

	var p2 map[int]string = map[int]string{}
	p2[1] = "Tom"
	fmt.Println("p2 :", p2)

	var p3 map[int]string = make(map[int]string)
	p3[1] = "Tom"
	fmt.Println("p3 :", p3)

	p4 := map[int]string{}
	p4[1] = "Tom"
	fmt.Println("p4 :", p4)

	p5 := make(map[int]string)
	p5[1] = "Tom"
	fmt.Println("p5 :", p5)
	
	p6 := map[int]string{
		1 : "Tom",
	}
	fmt.Println("p6 :", p6)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run declareMap.go
p1 : map[1:Tom]
p2 : map[1:Tom]
p3 : map[1:Tom]
p4 : map[1:Tom]
p5 : map[1:Tom]
p6 : map[1:Tom]
```

## 生成 JSON

```go
// mapToJson.go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	res := make(map[string]interface{})
	res["code"] = 200
	res["msg"]  = "success"
	res["data"] = map[string]interface{}{
		"username" : "Tom",
		"age"      : "30",
		"hobby"    : []string{"读书","爬山"},
	}
	fmt.Println("map data :", res)

	//序列化
	jsons, errs := json.Marshal(res)
	if errs != nil {
		fmt.Println("json marshal error:", errs)
	}
	fmt.Println("")
	fmt.Println("--- map to json ---")
	fmt.Println("json data :", string(jsons))

	//反序列化
	res2 := make(map[string]interface{})
	errs = json.Unmarshal([]byte(jsons), &res2)
	if errs != nil {
		fmt.Println("json marshal error:", errs)
	}
	fmt.Println("")
	fmt.Println("--- json to map ---")
	fmt.Println("map data :", res2)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run mapToJson.go
map data : map[code:200 data:map[age:30 hobby:[读书 爬山] username:Tom] msg:success]

--- map to json ---
json data : {"code":200,"data":{"age":"30","hobby":["读书","爬山"],"username":"Tom"},"msg":"success"}

--- json to map ---
map data : map[code:200 data:map[age:30 hobby:[读书 爬山] username:Tom] msg:success]
```

## 编辑和删除

```go
// mapDemo.go
package main

import (
	"fmt"
)

func main() {
	person := map[int]string{
		1 : "Tom",
		2 : "Aaron",
		3 : "John",
	}
	fmt.Println("data :",person)

	delete(person, 2)
	fmt.Println("data :",person)

	person[2] = "Jack"
	person[3] = "Kevin"
	fmt.Println("data :",person)
}
```

运行结果：

```shell
PS D:\Project\godemo> go run mapDemo.go
data : map[1:Tom 2:Aaron 3:John]
data : map[1:Tom 3:John]
data : map[1:Tom 2:Jack 3:Kevin]
```


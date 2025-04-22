---
title: "Golang学习日记Day1"
description: 本文记录了我今天学习的内容，主要包括包、变量、函数、控制结构、指针、数组、切片、map等基本语法知识。
date: 2025-04-22T16:43:16Z
image: go.jpg
categories:
    - Golang
---
## 包、变量与函数
### 包（Package）
Go 程序由包组成。每个 Go 源文件都需要以 package 声明开头。常见的 main 包表示可独立运行的程序，其入口为 main() 函数。

### 导入（Import）
通过 import 关键字导入其他包，例如：

```go
import "fmt"
```

### 导出名（Exported Name）
在 Go 中，以大写字母开头的标识符是可导出的（即公有），小写开头的则是私有的。

### 函数（Function）
函数使用 func 关键字定义，例如：

```go
func add(a int, b int) int {
    return a + b
}
```

### 多返回值
Go 支持一个函数返回多个值：

```go
func swap(x, y string) (string, string) {
    return y, x
}

```
### 带名字的返回值
返回值可以具名：

```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}
```

## 变量
### 声明变量
使用 var 声明变量：
```go
var i int
```
### 初始化变量
可以同时声明并初始化：
```go
var j int = 1
```
### 短变量声明
函数内部可以使用更简洁的 := 方式：
```go
k := 3
```

### 基本类型
Go 提供诸如 int, float64, bool, string 等基本类型。

### 零值
未显式初始化的变量会被赋予“零值”。

### 类型转换与类型推断
Go 不支持隐式类型转换，必须显式转换：

```go
var i int = 42
var f float64 = float64(i)
```

Go 具有类型推断能力：

```go
v := 42 // 推断为 int 类型
```

### 常量
使用 const 定义常量：

```go

const Pi = 3.14
```
## 控制结构
### for 循环
Go 仅有一种循环结构：for。

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```
#### 无限循环
```go
for {
    // ...
}
```
#### for 是 Go 中的 while
可用 for 模拟 while：
```go
for condition {
    // ...
}
```
### 条件语句
#### if 判断
```go
if x < 0 {
    x = -x
}
```
#### if 与简短语句
```go
if v := math.Pow(x, n); v < lim {
    return v
}
```
#### if-else
```go
if x < y {
    return x
} else {
    return y
}
```
### switch 分支
```go
switch i {
case 0:
    fmt.Println("Zero")
case 1:
    fmt.Println("One")
default:
    fmt.Println("Other")
}
```
#### 求值顺序
Go 的 switch 自动在 case 匹配成功后退出，无需 break。

#### 无条件 switch
```go
switch {
case x > 0:
    fmt.Println("Positive")
case x < 0:
    fmt.Println("Negative")
default:
    fmt.Println("Zero")
}
```
## defer 延迟调用
defer 会在函数返回前调用指定的语句，常用于资源释放。

```go
func main() {
    defer fmt.Println("world")
    fmt.Println("hello")
}
```
调用顺序为后进先出（LIFO）

## 指针与结构体
### 指针
```go
var p *int
p = &i
```
### 结构体
```go
type Vertex struct {
    X int
    Y int
}
```
### 结构体字段与指针
```go
v := Vertex{1, 2}
p := &v
p.X = 1e9
```

### 结构体字面量
```go
Vertex{X: 1, Y: 2}
```
## 数组与切片
### 数组
```go
var a [2]string
```
### 切片
```go
s := []int{1, 2, 3}
```
#### 切片类数组的引用特性
切片是对底层数组的引用，多个切片可共享底层数据。

#### 切片默认行为
```go
s[lo:hi]
```
长度与容量
```go
len(s), cap(s)
```
nil 切片与 make 创建
```go
var s []int // nil
s := make([]int, 5)
```
向切片添加元素
```go
s = append(s, 6)
```
range 遍历
```go
for i, v := range s {
    fmt.Println(i, v)
}
```
## Map 映射
### 定义与初始化
```go
m := make(map[string]int)
m["Answer"] = 42
```
### 映射字面量
```go
m := map[string]int{
    "Alice": 25,
    "Bob":   30,
}
```
### 修改、删除键值对
```go
delete(m, "Bob")
value, ok := m["Alice"]
```
## 函数进阶
### 函数值

```go
hypot := func(x, y float64) float64 {
    return math.Sqrt(x*x + y*y)
}
```

### 函数闭包
```go
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}
```
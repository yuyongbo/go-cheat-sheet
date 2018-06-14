# Go 备忘单

# 索引


1. [基础语法](#基础语法)
2. [运算符](#运算符)
   - [算术运算符](#算术运算符)
   - [比较运算符](#比较运算符)
   - [逻辑运算符](#逻辑运算符)
   - [其他](#其他)
3. [声明](#声明)
4. [函数](#函数)
   - [作为值和闭包的函数](#作为值和闭包的函数)
   - [可变参数函数](#可变参数函数)
5. [内置类型](#内置类型)
6. [类型转换](#类型转换)
7. [包](#包)
8. [控制结构](#控制结构)
   - [If条件](#条件)
   - [循环](#循环)
   - [Switch](#switch)
9. [数组、切片、范围 ](#数组、切片、范围 )
   - [数组](#数组)
   - [切片](#切片)
   - [操作数组和切片](#操作数组和切片)
10. [映射](#映射)
11. [结构体](#结构体)
12. [指针](#指针)
13. [接口](#接口)
14. [嵌入组合](#嵌入组合)
15. [错误](#错误)
16. [并发](#并发)
    - [协程](#协程)
    - [通道](#通道)
    - [通道原理](#channel的原理)
17. [打印输出](#打印)
18. [代码片段](#代码片段)
    - [HTTP服务器](#HTTP服务器)



## 引言

大部分的例子引用于 [A Tour of Go](http://tour.golang.org/)，非常棒的Go语言介绍。

如果你是Go的新手，请认真阅读Tour。

## Go核心技术

- 强类型语言
- 静态类型
- 语法类似于C (更少的括号和分号) 和 Oberon-2的struct结构体
- 编译为本地代码(没有java虚拟机)
- 没有类,但有带方法的结构
- 接口
- 没有实现继承，只有内嵌组合就够了
- 函数是一等公民
- 函数可以返回多值
- 有闭包
- 有指针，但没有指针的运算
- 内置的并发原语:协程和通道

# 基础语法

## 你好，世界

文件 `hello.go`:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello Go")
}
```

`$ go run hello.go`

## 运算符

### 算术运算符 

| 运算符号 | 描述   |
| -------- | ------ |
| `+`      | 加     |
| `-`      | 减     |
| `*`      | 乘     |
| `/`      | 商     |
| `%`      | 取余   |
| `&`      | 与     |
| `\|`     | 或     |
| `^`      | 位异或 |
| `&^`     | 位与非 |
| `<<`     | 左移   |
| `>>`     | 右移   |

### 比较运算符

| 运算符 | 描述                  |
| ------ | --------------------- |
| `==`   | 等于                  |
| `!=`   | 不等于                |
| `<`    | 小于                  |
| `<=`   | 小于等于    |
| `>`    | 大于          |
| `>=`   | 大于等于 |

### 逻辑运算符

| 运算符 | 描述   |
| ------ | ------ |
| `&&`   | 逻辑与 |
| `\|\|` | 逻辑或 |
| `!`    | 逻辑非 |

### 其他

| 运算符 | 描述                                         |
| ------ | -------------------------------------------- |
| `&`    | 对象地址 / 创建指针对象                      |
| `*`    | 指针解引用                                   |
| `<-`   | 发送 / 接收 运算符 (可查看文章下面的Channel) |

## 声明

具体类型需要写在识别符之后!

```go
var foo int // 声明未初始化
var foo int = 42 // 声明并初始化
var foo, bar int = 42, 1302 // 一次声明并初始化多个遍历
var foo = 42 // 省略类型, 自动推断类型
foo := 42 // 速写方式,只在函数内部,省略var书写，类型常是隐式的 
const constant = "This is a constant"
```

## 函数

```go
// 一个简单的函数
func functionName() {}

// 带参数的函数 (再次提示, 类型放在标识符后面)
func functionName(param1 string, param2 int) {}

// 带同一类型参数的函数
func functionName(param1, param2 int) {}

// 返回类型声明
func functionName() int {
    return 42
}

// 一次能返回多个值
func returnMulti() (int, string) {
    return 42, "foobar"
}
var x, str = returnMulti()

// 返回多个已命名过的值可以直接简化成return 
func returnMulti2() (n int, s string) {
    n = 42
    s = "foobar"
    // n和s会被return
    return
}
var x, str = returnMulti2()

```

### 作为值和闭包的函数

```go
func main() {
    // 赋值函数给一个变量
    add := func(a, b int) int {
        return a + b
    }
    // 用变量名字调用函数
    fmt.Println(add(3, 4))
}

// 闭包，词法作用域：其他函数可以访问在scope函数里定义的值
func scope() func() int{
    outer_var := 2
    foo := func() int { return outer_var}
    return foo
}

func another_scope() func() int{
    // 不会编译，因为outer_var和foo没有在这个作用域内声明
    outer_var = 444
    return foo
}


// 闭包：不会改变函数外的变量，而是重新声明它们！
func outer() (func() int, int) {
    outer_var := 2
    inner := func() int {
        outer_var += 99 // 试图在作用域中改变outer_var
        return outer_var // => 101 (outter_var是重新声明的变量
                         //         变量只能在函数内部可使用)
    }
    return inner, outer_var // => 101, 2 (outer_var 一直是2, 没有被闭包内部改变!)
}
```

### 可变参数函数

```go
func main() {
	fmt.Println(adder(1, 2, 3)) 	// 6
	fmt.Println(adder(9, 9))	// 18
	
	nums := []int{10, 20, 30}
	fmt.Println(adder(nums...))	// 60
}

// 使用 ... 可以在最后一个参数的类型名之前替代0个或更多的参数。
// 这个函数跟其他函数一样可以被调用并且可以传递更多的参数
func adder(args ...int) int {
	total := 0
	for _, v := range args { // Iterates over the arguments whatever the number.
		total += v
	}
	return total
}
```

## 内置类型

```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8的别名

rune // int32的别名 ~= 一个字符（Unicode编码点）-非常(霸道)Viking

float32 float64

complex64 complex128
```

## 类型转换

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)

// 可选的语法规则
i := 42
f := float64(i)
u := uint(f)
```

## 包

- 包声明在每个源文件的头部
- 可执行文件在 `main`包里
- 约定: 包名 == import路径的最后一个词名
- 大写字母标识符: 公开的 (外部包可引用的)
- 小写字母标识符: 私有的(外部包不可引用的) 

## 控制结构

### 条件

```go
func main() {
	// 基础的
	if x > 0 {
		return x
	} else {
		return -x
	}
    	
	// 可以在条件之前放一个表达式语句
	if a := b + c; a < 42 {
		return a
	} else {
		return a - 42
	}
    
	// if内部的类型断言
	var val interface{}
	val = "foo"
	if str, ok := val.(string); ok {
		fmt.Println(str)
	}
}
```

### 循环

```go
// 这里只有for，没有while，until
for i := 1; i < 10; i++ {
}
for ; i < 10;  { // while - 循环
}
for i < 10  { // 如果只有一个条件的话可以省略分号；
}
for { // 可以省略条件 ~ while (true)
}
```

### Switch

```go
// switch 语句
switch operatingSystem {
case "darwin":
    fmt.Println("Mac OS Hipster")
    // cases会自动break, 而不是通过defaultbreak
case "linux":
    fmt.Println("Linux Geek")
default:
    // Windows, BSD, ...
    fmt.Println("Other")
}

// 与for和if一样，可以在switch值之前加一个赋值语句
switch os := runtime.GOOS; os {
case "darwin": ...
}

// 可以在 switch cases中做比较
number := 42
switch {
    case number < 42:
        fmt.Println("Smaller")
    case number == 42:
        fmt.Println("Equal")
    case number > 42:
        fmt.Println("Greater")
}

// 在case列表中可以用多个用逗号分隔
var char byte = '?'
switch char {
    case ' ', '?', '&', '=', '#', '+', '%':
        fmt.Println("Should escape")
}    
```

## 数组、切片、范围 

### 数组

```go
var a [10]int // 声明了一个长度为10 的int数组，数组长度是类型的一部分
a[3] = 42     // 设置元素的值
i := a[3]     // 读取元素

// 声明并初始化
var a = [2]int{1, 2}
a := [2]int{1, 2} //简写法
a := [...]int{1, 2} // ...省略号 -> 编译器计算出数组长度
```

### 切片

```go
var a []int                              // 声明一个切片，-类似与一个数组，但是长度为指明
var a = []int {1, 2, 3, 4}               // 声明并初始化一个切片（背后是底层隐式的数组）
a := []int{1, 2, 3, 4}                   // 简写法
chars := []string{0:"a", 2:"c", 1: "b"}  // ["a", "b", "c"]

var b = a[lo:hi]	// 创建一个切片（可看成数组）索引从lo到hi
var b = a[1:4]		// 索引从1到3的切片
var b = a[:3]		// 省略前面的索引意指0
var b = a[3:]		// 省略后面的索引意指len(a)
a =  append(a,17,3)	//  在切片a后面追加item
c := append(a,b...)	//  连接切片a和b

// 用make创建切片
a = make([]byte, 5, 5)	// 第一个参数代表长度，第二个代表容量
a = make([]byte, 5)	// 容量是可选参数

// 从数组里面创建切片
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // 一个切片引用了x的存储容量
```

### 操作数组和切片

`len(a)`可以算出一个数组或切片的长度，它是内建函数，而不是一个数组的属性/方法

```go
// 对一个数组/一个切片的循环
for i, e := range a {
    // i 是索引, e 是元素
}

// 如果只要e:
for _, e := range a {
    // e 是元素
}

// 如果只需要索引
for i := range a {
}

// 在Go1.4之前，如果你不使用i和e，会得到一个编译错误。
// Go1.4引入了一种无变量形式，就可以这样使用
for range time.Tick(time.Second) {
    // do it once a sec
}

```

## 映射

```go
var m map[string]int
m = make(map[string]int)
m["key"] = 42
fmt.Println(m["key"])

delete(m, "key")

elem, ok := m["key"] // 再次测试key是否存在

// 字符组合的map
var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}

```

## 结构体

这里没有类，只有结构体，结构体可以有方法

```go
// 结构体是一种类型. 代表字段的集合

// 声明
type Vertex struct {
    X, Y int
}

// 创建
var v = Vertex{1, 2}
var v = Vertex{X: 1, Y: 2} // 通过key-value创建一个结构体实体
var v = []Vertex{{1,2},{5,2},{5,5}} // 初始化结构体 切片

// 访问字段
v.X = 4

// 您可以在struct中声明方法。想要声明的结构体方法（接收类型）在func关键字和
// 方法名之间。结构体会复制在每个方法（）调用之前
func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// 调用方法
v.Abs()

// 对于改变变量的方法，需要通过使用结构体指针作为类型。这样在调用过程中结构体的值将不会被拷贝【所谓的值拷贝】
func (v *Vertex) add(n float64) {
    v.X += n
    v.Y += n
}

```

**匿名结构体:**  
比起用 `map[string]interface{}`成本更低更安全

```go
point := struct {
	X, Y int
}{1, 2}
```

## 指针

```go
p := Vertex{1, 2}  // p 是一个 Vertex
q := &p            // q 是指向 Vertex的一个指针
r := &Vertex{1, 2} // r 也是指向 Vertex的一个指针

//  Vertex 的指针类型是 *Vertex

var s *Vertex = new(Vertex) // new 创建了一个指针指向了新的结构体实体 
```

## 接口

```go
// 接口声明
type Awesomizer interface {
    Awesomize() string
}

// 类型没有声明实现的接口
type Foo struct {}

// 相反，如果它们实现了所有必需的方法，类型就会隐式地满足接口
func (foo Foo) Awesomize() string {
    return "Awesome!"
}
```

## 嵌入组合

在Go中没有子类，而是interface和struct的内嵌组合。

```go
// ReadWriter 的实现必须满足Reader和Writer
type ReadWriter interface {
    Reader
    Writer
}

// Server 公开了Logger所有的方法
type Server struct {
    Host string
    Port int
    *log.Logger
}

// 按照通常的方式初始化组合类型
server := &Server{"localhost", 80, log.New(...)}

// 可以使用在嵌入式结构中实现的方法
server.Log(...) // 调用 server.Logger.Log(...)

// 嵌入式结构的字段名是它的类型名称（在Logger中）
var logger *log.Logger = server.Logger
```

## 错误

没有异常处理 . 可能产生错误的函数只是声明了 `Error`类型的返回值. 下面是 `Error` 的interface:

```go
type error interface {
    Error() string
}
```

下面方法可能会返回一个err:

```go
func doStuff() (int, error) {
}

func main() {
    result, err := doStuff()
    if err != nil {
        // 处理错误
    } else {
        // 无错误，使用正确结果
    }
}
```

# 并发

## 协程

Goroutines 是轻量级的线程 (被Go操纵管理，而不是操作系统线程). `go f(a, b)` 开启了一个新的运行f函数的goroutine (`f`是个函数).

```go
// 只是一个函数（会被作为一个goroutine开始执行）
func doStuff(s string) {
}

func main() {
    // 用一个已命名的函数开启goroutine
    go doStuff("foobar")

    // 用一个匿名开启goroutine
    go func (x int) {
        // 方法体放在这里
    }(42)
}
```

## 通道

```go
ch := make(chan int) // 创建一个int类型的channel
ch <- 42             // 给channel ch发送一个值
v := <-ch            // 从ch接受一个值

// 无缓冲的channle会阻塞. 没有值可读的时候会阻塞，写也是，直到有值可读。

// 创建一个带缓冲的channel，如果写的未读值小于buffer大小，那么写入缓冲通道就不会阻塞。
ch := make(chan int, 100)

close(ch) // 关闭通道（只有发送方能关闭）

// 如果channel已经关闭，可以从通道中读取和测试
v, ok := <-ch

// 如果ok是false，说明channel已经关闭

// 一直从channel里读直到关闭
for i := range ch {
    fmt.Println(i)
}

// 在多个通道操作上select ，如果一个没有阻塞，与之对应的case将被执行
func doStuff(channelOut, channelIn chan int) {
    select {
    case channelOut <- 42:
        fmt.Println("We could write to channelOut!")
    case x := <- channelIn:
        fmt.Println("We could read from channelIn")
    case <-time.After(time.Second * 1):
        fmt.Println("timeout")
    }
}
```

### Channel的原理

- 给一个nil的channel发送永远会阻塞

  ```go
  var c chan string
  c <- "Hello, World!"
  // fatal error: all goroutines are asleep - deadlock!
  ```

- 从一个nil的channel接受也会永远阻塞

  ```go
  var c chan string
  fmt.Println(<-c)
  // fatal error: all goroutines are asleep - deadlock!
  ```

- 给一个已关闭的channel发送会panic

  ```go
  var c = make(chan string, 1)
  c <- "Hello, World!"
  close(c)
  c <- "Hello, Panic!"
  // panic: send on closed channel
  ```

- 从一个关闭的channel接受会立刻返回一个0值

  ```go
  var c = make(chan int, 2)
  c <- 1
  c <- 2
  close(c)
  for i := 0; i < 3; i++ {
      fmt.Printf("%d ", <-c) 
  }
  // 1 2 0
  ```

## 打印

```go
fmt.Println("Hello, 你好, नमस्ते, Привет, ᎣᏏᏲ") // 基本的 print, 会自动换行
p := struct { X, Y int }{ 17, 2 }
fmt.Println( "My point:", p, "x coord=", p.X ) // print 结构体，整型等等
s := fmt.Sprintln( "My point:", p, "x coord=", p.X ) // print 字符串变量

fmt.Printf("%d hex:%x bin:%b fp:%f sci:%e",17,17,17,17.0,17.0) // c-ish 格式化
s2 := fmt.Sprintf( "%d %f", 17, 17.0 ) // 格式化输出字符串变量

hellomsg := `
 "Hello" in Chinese is 你好 ('Ni Hao')
 "Hello" in Hindi is नमस्ते ('Namaste')
` // 多行的字符串, 用``
```

# 代码片段

## HTTP服务器

```go
package main

import (
    "fmt"
    "net/http"
)

// 定义了一个响应体struct
type Hello struct{}

// 让Hello去实现ServeHTTP 方法 (在 interface http.Handler里面已经定义)
func (h Hello) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello!")
}

func main() {
    var h Hello
    http.ListenAndServe("localhost:4000", h)
}

// 这里是 http.ServeHTTP的方法署名:
// type Handler interface {
//     ServeHTTP(w http.ResponseWriter, r *http.Request)
// }
```
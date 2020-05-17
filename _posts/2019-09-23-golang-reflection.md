---
layout:     post
title:      "如何理解 Go 中的反射"
subtitle:   "The Laws of Reflection"
date:       2019-09-23
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-hadoop-01.jpg"
categories: [Go]
tags:
    - Go
---

本文概览：
* TOC
{:toc}

> 首先给大家推荐一个在线 `Go` 运行环境，可以测试简短的代码逻辑。[https://play.studygolang.com](https://play.studygolang.com)

Go 中的反射是基于类型（`type`）机制的，所以需要重温一下 `Go` 中的类型机制。

## 1. Types and interfaces
Go 是静态类型语言。 每个变量都有一个静态类型，也就是在编译时已知并固定的一种类型：`int，float32，*MyType，[]byte` 等。 如果我们声明：
```go
type MyInt int

var i int
var j MyInt
```
则变量 `i` 是 `int` 类型，变量 `j` 是 `MyInt` 类型。变量 `i` 和 `j` 具有不同的静态类型，尽管它们具有相同的基础类型，但是如果不进行转换依然无法将其中一个变量赋值于另一个变量。

Go 中一个重要的类别是接口类型（interface），接口表示固定的方法集。接口变量可以存储任何具体的（非接口）值，只要该值实现了接口中所有定义的方法即可。 一个重要的例子就是`io.Reader`与`io.Writer`， 类型 `Reader` 与 `Writer` 都来自 [io - The Go Programming Language](https://golang.org/pkg/io/) 包
```go
// Reader is the interface that wraps the basic Read method.
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
任何只要实现了 `Read` 或者 `Write` 方法的类型都算作实现了 `io.Reader` 或者 `io.Writer` 接口，这意味着 `io.Reader` 类型的变量可以保存其类型具有 `Read` 方法的任何值：
```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```
重要的是要清楚，无论 `r` 可能包含什么具体值，`r` 的类型始终是 `io.Reader`：Go是静态类型的，而 `r` 的静态类型是`io.Reader`。

接口类型的一个非常重要的例子是空接口：
```go
interface{}
```
它表示空方法集，并且任何值都满足实现了空接口，因为任何值具有零个或多个方法，而空接口没有方法供实现。

有人说 `Go` 的空接口是动态类型的，但这会产生误导。它们是静态类型的：接口类型的变量始终具有相同的静态类型，即使在运行时存储在接口变量中的值可能会更改类型，但该值也还是始终满足接口的要求。

## 2. The representation of an interface
接口类型的变量存储一对儿信息，分别是分配给该变量的具体值以及该值的类型描述符。
例如：
```go
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```
在变量 `r` 中则存储了 `(value, type) ` 对，内容为 `(tty, *os.File)`。值得注意的是，即使接口变量 `r` 仅提供对 `Read` 方法的访问，但内部的值仍包含有关该值的所有类型信息。所以下面这个代码也是正确的：
```go
var w io.Writer
w = r.(io.Writer)
```
这个赋值操作中的表达式是类型断言。它断言 `r` 内的项也实现了 `io.Writer`，因此我们可以将其分配给接口变量 `w`。赋值后，`w` 也同样包含一对信息 —— `(tty，* os.File)`。接口的静态类型会决定使用接口变量调用哪些方法，即使内部的具体值可能具有更大的方法集。

强调一遍，在一个接口变量中一直都是保存一对信息，格式为 `(value, concrete type)`，但是不能保存 `(value, interface type)` 格式。
> 在 Go 语言中，变量类型分为两大类，`concrete type` 与 `interface type`{
> concrete type: 指具体的变量类型，可以是基本类型，也可以是自定义类型或者结构体类型；
> interface type: 指接口类型，可以是 Go 内置的接口类型，或者是使用者自定义的接口类型；}

而之所以先重温接口就是因为反射和接口息息相关

## 3. Three law of reflection
### 3.1. Reflection goes from interface value to reflection object.
从底层层面来说，反射是一种解释存储在接口类型变量中的 `(type, value)` 一对信息的机制。首先，我们需要在反射包中了解两种类型：`type` 和 `value`，通过这两种类型对接口变量内容的访问，还有两个对应的函数，称为 `reflect.TypeOf` 和`reflect.ValueOf`，从接口值中获取 `reflect.Type` 和 `reflect.Value` 部分。
例如 `TypeOf`：
```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
    fmt.Println("value:", reflect.ValueOf(x))
}
```
结果输出为：
```go
type: float64
value: 3.4
```
说明：
- reflect.TypeOf：获得值的类型（`type`），如 `float64、int、pointer、struct` 等等真实的类型；
- reflect.ValueOf：获得值的内容，如1.2345这个具体数值，或者类似 `&{1 “Allen.Wu” 25}` 这样的结构体 struct 的内容；
- 说明反射可以将“接口类型变量”转换为“反射类型对象”，反射类型指的是 `reflect.Type` 和 `reflect.Value` 这两个函数的返回；

`reflect.TypeOf` 的函数签名包括一个空接口：
```go
// TypeOf returns the reflection Type of the value in the interface{}.
func TypeOf(i interface{}) Type
```
当我们调用 `reflect.TypeOf(x)`时，`x` 首先存储在一个空接口中，然后将其作为参数传递； `reflect.TypeOf` 解压缩该空接口以恢复类型信息。
又例如：
```go
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x))
fmt.Println("value:", reflect.ValueOf(x).String())
```
输出结果为：
```go
value: 3.4
value: <float64 Value>
```

`reflect.Type` 和 `reflect.Value` 都有很多方法可以让我们检查和操作它们。 一个重要的例子是 `Value` 具有 `Type` 方法，该方法返回 `reflect.Value` 的 `Type`。另一个是 `Type` 和 `Value` 都有 `Kind` 方法，该方法返回一个常量，指示存储的项目类型：`Uint，Float64，Slice` 等。

反射库具有几个值得一提的属性。

首先，为使 API 保持简单，`Value` 的 `“getter”` 和 `“setter”` 方法在可以容纳该值的最大类型上运行：例如，所有有符号整数的 `int64`。也就是说，`Value` 的 `Int` 方法返回一个 `int64`，而 `SetInt` 值采用一个 `int64`，可能需要转换为涉及的实际类型：
```go
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())
```

第二个属性是反射对象的 `Kind()` 方法描述基础类型，而不是静态类型。例如：
```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    type MyInt int		// 反射对象包含用户定义的整数类型的值
    var x MyInt = 7
    v := reflect.TypeOf(x)
    fmt.Println(v)
    fmt.Println(v.Kind())
}
```
则会输出：
```go
main.MyInt
int
```

### 3.2. Reflection goes from reflection object to interface value.
Go 的反射也有其逆向过程。

给定一个 `reflect.Value` ，我们可以使用 `Interface()` 方法恢复接口值，该方法将 `type` 和 `value` 信息打包回接口表示形式并返回结果：
```go
// Interface returns v's value as an interface{}.
func (v Value) Interface() interface{}
```
例如：
```go
func main() {
    var xx float64 = 3.4
    v := reflect.ValueOf(xx)     // v is a reflection object
    y := v.Interface().(float64) // y will have type float64.
    fmt.Println(y)
    fmt.Printf("%T", y)
}
```
输出结果为：
```go
3.4
float64
```

简而言之，`Interface` 方法与 `ValueOf` 函数相反，但其结果始终是静态类型 `interface{}`。

所以综上述两点可得知，Go 中的反射可理解为包含两个过程，一个是接口值到反射对象的过程，另一个则是反向的反射对象到接口值的过程。

### 3.3. To modify a reflection object, the value must be settable.
第三条规律则是如果想要修改一个反射对象（`reflection object`），那么这个对象的值必须是**可设置的**。直接这样说会比较困惑，从例子出发：
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```
如果运行上述这个代码，则会报错提示:
```go
panic: reflect: reflect.Value.SetFloat using unaddressable value
```
在这个例子中，反射对象 v 的值就是不可设置的，执行下述代码：
```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
```
则会显示：
```go
settability of v: false
```

那么什么是可设置的呢，在 Go 官网原文有这么一句

> Settability is determined by whether the reflection object holds the original item.

翻译过来就是可设置性由反射对象是否保留原始对象确定。我们都知道在 Go 中的参数传递都是使用的值传递的方法，即将原有值的拷贝传递，在刚刚的例子中，我们是传递了一个 `x` 对象的拷贝到 `reflect.ValueOf` 函数中，而不是 `x` 对象本身，刚刚的 `SetFloat ` 将更新存储在反射对象内的 `x` 的副本，并且 `x`本身将不受影响，在 Go 中这是不合理的，可设置性就是避免此问题的属性。

而如果我们想要修改其内容，很简单，将对象的指针传入其中，于是刚刚的代码可以改为：
```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())

v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
fmt.Println("----------------")

v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(z)
```
此时输出：
```go
float64type of p: *float64
settability of p: false
settability of v: true
----------------
7.1
7.1
```

#### Reflection and Structs

反射修改内容一个经常使用的地方就是通过指针修改传入的结构体的字段值，只要我们能够获得该结构体对象的指针。

一个简单的示例。
```go
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i, typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```
这里使用结构的地址创建了反射对象，然后稍后将要对其进行修改。将 `typeOfT` 设置为其类型，并使用简单的方法调用对字段进行迭代。请注意，我们从结构类型中提取了字段的名称，但是字段本身是常规的 `reflect.Value` 对象。这里结果输出为：
```go
0: A int = 23
1: B string = skidoo
```
> 这里有一点要注意的是，结构体 `T` 的字段名首字母都是大写，在 Go 中首字母大写的变量或者函数才是可导出的（exported），相当于 Java 中的 `public`，而首字母小写的变量或者函数则是包外不可使用，对应 Java 的 `protected`。 而只有可导出的结构体字段此方式才能修改。

现在我们可以试着修改结构体 `T`：
```go
s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)

// output is "t is now {77 Sunset Strip}"
```

## 4. Conclusion
反射的三条规律：
- 反射包括从接口值到反射对象的过程；
- 反射也包括从反射对象到接口值的过程；
- 要修改反射对象，该值必须可设置（`To modify a reflection object, the value must be settable.`）。


**参考文献**
- [research!rsc: Go Data Structures: Interfaces](https://research.swtch.com/interfaces)
- [The Laws of Reflection - The Go Blog](https://blog.golang.org/laws-of-reflection)

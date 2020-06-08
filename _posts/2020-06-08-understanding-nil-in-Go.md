---
layout:     post
title:      "1 分钟阅读系列--Understanding nil in Go"
subtitle:   "Understanding nil in Go"
date:       2020-06-08
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-nilGo.jpg"
categories: [Go]
tags:
    - 1分钟阅读系列
    - Go
---

在 Go 语言中，`nil` 表示零值（zero value），Go 语言中的每个类型的都有零值，`int` 类型的零值是 0，`float` 类型的零值是 0.0，`boolean` 类型的零值是 false，`string` 类型的零值是空字符串 `""`，而其他类型，诸如 `pointer`、`slice`、`map`、`channel`、`function`、`interface` 等类型的零值则是 nil。

> nil is a predeclared identifier representing the zero value for a pointer, channel, func, interface, map, or slice type.

![各种类型的零值](https://live.staticflickr.com/65535/49983958898_dec2575939_o.png)

而对于一个结构体来说，零值则是其成员的零值，例如：

```go
type Person struct {
    AgeYears int
    Name     string
    Friend   []Person
}

var p Person
```

那么变量 `p` 的零值则是 `Person{0, "", nil}`。

虽然 nil 对于不同类型来说表达的都是零值，但是具体情况是不尽相同：对于 `pointer` 指针类型，nil 表示给指针不指向任何变量；而 `slice` 切片类型包含三个部分，即 `{ptr, len, cap}`，分别表示「指向底层数组的指针，切片长度，切片容量」，所以 slice 切片类型的零值更深层次则是 {nil, 0, 0}，但是在代码层面依然可以用 `s == nil` 进行判断。

对于 `interface` 接口类型，事情则会变得更加复杂一些。首先一个 interface 类型变量由两部分组成：**类型** 以及 **值**，即 `(type, value)`，例如：

```go
var s fmt.Stringer     // Stringer(nil, nil)
fmt.Println(s == nil)  // true
```

当我们只是定义一个接口类型变量时，该变量就是一个 type 以及 value 都是 nil 的接口变量，此时 `s == nil` 判断得到的则为 true。

但如果我们给一个接口类型变量赋予一个类型时，此时就发生了改变，例如：

```go
var p *Person             // p is nil of type *Person
var s fmt.Stringer = p    // Stringer(*Person, nil)
fmt.Println(s == nil)     // false
```

此时接口变量的类型则变成了 `(*Person, nil)`，此时 `s == nil` 判断的则是 false，证明当接口类型变量的 type 不为 nil 时，此变量就不为 nil。

> when interfaces are nil: have no value assigned, not even a nil pointer

OK，由此我们可以继续引出一个常见的失误点。在 Go 语言中没有诸如 `try-catch` 这样的异常捕捉，对于函数的错误都是采取返回的方式，然后在函数调用处采用 `err == nil` 的方式来判断是否产生错误。然而有时我们需要自定义错误时，则会产生如下写法：

```go
func do() error {           // error(*MyError, nil)
    var err *MyError
    return err              // err is nil of type *MyError
}

func main() {
    err := do()             // error(*MyError, nil)
    fmt.Println(err)        // <nil>
    fmt.Println(err == nil) // false
}
```

因为返回的是我们自定义的错误类型，当我们再通过 error 接口类型包装后返回时，返回的错误变量 err 的类型实际已变成了我们自定义的 `*MyError`，在函数调用处的 `err == nil` 则会为 false，但是当你打印出 err 时却显示依然是 `<nil>`，**这就是 Go 语言编程中典型的 nil 不等于 nil 的错误**。所以保险起见，我们最好不要自定义err 变量。那么如果需要自定义 err 类型时，应该如何写呢？我们只需要将函数返回声明由 error 接口类型变为指针类型即可：

```go
func do() *MyError {        // nil of type *MyError
    return nil
}

func main() {
    err := do()
    fmt.Println(err)        // <nil>
    fmt.Println(err == nil) // true
}
```

此时我们返回的是一个指针变量，所以可以放心使用 `err == nil` 进行判断。


参考：
[GopherCon 2016: Francesc Campoy - Understanding nil](https://www.youtube.com/watch?v=ynoY2xz-F8s)

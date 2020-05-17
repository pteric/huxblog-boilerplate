---
layout:     post
title:      "Go 中的错误与异常"
subtitle:   "The error & exception in Go"
date:       2019-09-11
author:     "PengTuo"
catalog:    true
header-img: "img/post-bg-go.jpg"
categories: [Go]
tags:
    - Go
---

本文概览：
* TOC
{:toc}


在 Go 语言中，错误和异常从语言机制上面讲，就是 error 和 panic 的区别，放到别的语言也一样，别的语言没有 error 类型，但是有错误码之类的，没有panic，但是有 throw 之类的。

在语言层面它们是两种概念，错误指的是可能出现问题的地方出现了问题，比如打开一个文件时失败，这种情况在人们的意料之中；而异常指的是不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外。导致的是两种不同的结果。如果程序遇到错误不处理，那么可能进一步的产生业务上的错误；如果程序遇到异常不处理，那么结果就是进程异常退出。可以理解错误是业务过程的一部分，异常不是。

Go中，函数运行失败时会返回错误信息，这些错误信息被认为是一种预期的值而非异常（exception），这使得 Go 有别于那些将函数运行失败看作是异常的语言（例如 Java）。虽然 Go 有各种异常机制，但这些机制仅被使用在处理那些未被预料到的错误，即 bug，而不是那些在健壮程序中应该被避免的程序错误。

**什么情况下用错误表达，什么情况下用异常表达，就得有一套规则，否则很容易出现一切皆错误或一切皆异常的情况。**

## Go 中的错误处理
Go 使用控制流机制（如if和return）处理异常，这使得编码人员能更多的关注错误处理。当一次函数调用返回错误时，调用者有应该选择何时的方式处理错误。根据情况的不同，有很多处理方式，让我们来看看常用的几种方式。

### 1. 传播错误
即函数中某个子程序的失败，会变成该函数的失败。该函数直接将其内部某个子程序的 error 返回，
```go
resp, err := http.Get(url)
if err != nil{
    return nill, err
}
```
或者是根据内部子程序发生的错误构造一个新的错误返回给调用者，以助于错误的处理。
```go
doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url,err)
}
```
编写错误信息时，我们要确保错误信息对问题细节的描述是详尽的。尤其是要注意错误信息表达的一致性，即相同的函数或同包内的同一组函数返回的错误在构成和处理方式上是相似的。

以OS包为例，OS包确保文件操作（如os.Open、Read、Write、Close）返回的每个错误的描述不仅仅包含错误的原因（如无权限，文件目录不存在）也包含文件名，这样调用者在构造新的错误信息时无需再添加这些信息。

### 2. 操作失败重试
如果错误的发生是偶然性的，或由不可预知的问题导致的。一个明智的选择是重新尝试失败的操作。在重试时，我们需要限制重试的时间间隔或重试的次数，防止无限制的重试。
```go
// WaitForServer attempts to contact the server of a URL.
// It tries for one minute using exponential back-off.
// It reports an error if all attempts fail.
func WaitForServer(url string) error {
    const timeout = 1 * time.Minute
	   // 被调用时设置执行时长 1 分钟
    deadline := time.Now().Add(timeout)
    for tries := 0; time.Now().Before(deadline); tries++ {
        _, err := http.Head(url)
        if err == nil {
            return nil // success
        }
        log.Printf("server not responding (%s);retrying…", err)
        time.Sleep(time.Second << uint(tries)) // exponential back-off
    }
    return fmt.Errorf("server %s failed to respond after %s", url, timeout)
}
```

### 3. 输出错误程序
首先一种是输出错误程序并且中断程序，这种策略只应在main中执行。对库函数而言，应仅向上传播错误，除非该错误意味着程序内部包含不一致性，即遇到了bug，才能在库函数中结束程序。
```go
if err := WaitForServer(url); err != nil {
    log.Fatalf("Site is down: %v\n", err)
}
```
> 注意，log.Fatalf() 函数不仅会输出错误，还会中断程序

其次就是只需要输出错误信息就足够了，不需要中断程序的运行。我们可以通过log包提供函数
```go
if err := Ping(); err != nil {
    log.Printf("ping failed: %v; networking disabled",err)
}
```
或者标准错误流输出错误信息。
```go
if err := Ping(); err != nil {
    fmt.Fprintf(os.Stderr, "ping failed: %v; networking disabled\n", err)
}
```

### 4. 忽略错误
最后一种策略：我们可以直接忽略掉错误，但需要确定程序的逻辑不会因此受到影响。我们应该在每次函数调用后，都养成考虑错误处理的习惯，当你决定忽略某个错误时，你应该在清晰的记录下你的意图。

在Go中，错误处理有一套独特的编码风格。检查某个子函数是否失败后，我们通常将处理失败的逻辑代码放在处理成功的代码之前。如果某个错误会导致函数返回，那么成功时的逻辑代码不应放在else语句块中，而应直接放在函数体中。Go中大部分函数的代码结构几乎相同，首先是一系列的初始检查，防止错误发生，之后是函数的实际逻辑。

其次，在 Go 中对于错误的处理应该养成几种习惯：
1. **失败的原因只有一个时，不使用error，改为使用 bool**
2. **error应放在返回值类型列表的最后**
3. **错误值统一定义，而不是跟着感觉走**：不同的人构建一个项目时，会出现每个人错误的 value 值形式各不相同，当上层函数要对特定错误 value 进行统一处理时，需要漫游所有下层代码，以保证错误 value 统一，不幸的是有时会有漏网之鱼，而且这种方式严重阻碍了错误 value 的重构。我们可以参考C/C++的错误码定义文件，在 Go 的每个包中增加一个错误对象定义文件：
```go
var ERR_EOF = errors.New("EOF")
var ERR_CLOSED_PIPE = errors.New("io: read/write on closed pipe")
var ERR_NO_PROGRESS = errors.New("multiple Read calls return no data or error")
var ERR_SHORT_BUFFER = errors.New("short buffer")
var ERR_SHORT_WRITE = errors.New("short write")
var ERR_UNEXPECTED_EOF = errors.New("unexpected EOF")
```

4. **错误处理使用defer**


5. **当上层函数不关心错误时，建议不返回error**

##  Go 中的异常处理
Go 中引入两个内置函数 panic 和 recover 来触发和终止异常处理流程，同时引入关键字 defer 来延迟执行 defer 后面的函数。如果在 defer 函数中调用了内置函数 recover，并且定义该 defer 语句的函数发生了 panic 异常，recover 会使程序从 panic 中恢复，并返回 panic value。导致 panic 异常的函数不会继续运行，但能正常返回。在未发生 panic 时调用 recover，recover 会返回 nil。

由于 panic 会引起程序的崩溃，因此 panic 一般用于严重错误，如程序内部的逻辑不一致。对于大部分漏洞，我们应该使用 Go 提供的错误机制，而不是 panic，尽量避免程序的崩溃。

不加区分的恢复所有的panic异常，不是可取的做法；因为在panic之后，无法保证包级变量的状态仍然和我们预期一致。比如，对数据结构的一次重要更新没有被完整完成、文件或者网络连接没有被关闭、获得的锁没有被释放。此外，如果写日志时产生的panic被不加区分的恢复，可能会导致漏洞被忽略。

虽然把对panic的处理都集中在一个包下，有助于简化对复杂和不可以预料问题的处理，但作为被广泛遵守的规范，你不应该试图去恢复其他包引起的panic。公有的API 应该将函数的运行失败作为 error 返回，而不是 panic。同样的，你也不应该恢复一个由他人开发的函数引起的 panic，比如说调用者传入的回调函数，因为你无法确保这样做是安全的。

基于以上原因，安全的做法是有选择性的recover。换句话说，只恢复应该被恢复的panic异常，此外，这些异常所占的比例应该尽可能的低。为了标识某个panic是否应该被恢复，我们可以将panic value设置成特殊类型。在recover时对panic value进行检查，如果发现panic value是特殊类型，就将这个panic作为errror处理，如果不是，则按照正常的panic进行处理。

一般使用异常的场景有：
1. **在程序开发阶段，坚持速错**：在早期开发以及任何发布阶段之前，最简单的同时也可能是最好的方法是调用panic函数来中断程序的执行以强制发生错误，使得该错误不会被忽略，因而能够被尽快修复。

>我们在调用recover的延迟函数中以最合理的方式响应异常：
> a. 打印堆栈的异常调用信息和关键的业务信息，以便这些问题保留可见；
> b. 将异常转换为错误，以便调用者让程序恢复到健康状态并继续安全运行。

2. **在程序部署后，应恢复异常避免程序终止**
3. **对于不应该出现的分支，使用异常处理**
4. **针对入参不应该有问题的函数，使用panic设计**
5. **一些常见问题**：空指针引用，下标越界，除数为0 等

> 此外

抛开 Go 语言，从研发的角度来说处理异常也应该做到以下几点：
1. 不要随便 re-throw
就是 catch 错误再抛出，原因是异常的第一现场会被破坏，堆栈跟踪信息会丢失，因为外部最后拿到异常的堆栈跟踪信息，是最后那次throw的异常的堆栈跟踪信息。
2. try-catch 后一定要记录异常
首先，catch 操作很容易导致异常暴露不出来，升级为更严重的业务漏洞，但是我们也需要确保程序的健壮性，所以在 catch 到异常后需要及时记录。

---
title: Go 中常见错误
date: 2019-03-11 21:15:27
tags: [Go, Note]
categories: [Go]
permalink: Go-Common-Mistakes
---

# Tips  
- 永远不要使用形如 `var p*a` 声明变量，这会混淆指针声明和乘法运算（参考[4.9小节](04.9.md)）
- 永远不要在`for`循环自身中改变计数器变量（参考[5.4小节](05.4.md)）
- 永远不要在`for-range`循环中使用一个值去改变自身的值（参考[5.4.4小节](05.4.md)）
- 永远不要将`goto`和前置标签一起使用（参考[5.6小节](05.6.md)）
- 永远不要忘记在函数名（参考[第6章](06.0.md)）后加括号()，尤其调用一个对象的方法或者使用匿名函数启动一个协程时
- 永远不要使用`new()`一个map，一直使用make（参考[第8章](08.0.md)）
- 当为一个类型定义一个String()方法时，不要使用`fmt.Print`或者类似的代码（参考[10.7小节](10.7.md)）
- 永远不要忘记当终止缓存写入时，使用`Flush`函数（参考[12.2.3小节](12.2.md)）
- 永远不要忽略错误提示，忽略错误会导致程序奔溃（参考[13.1小节](13.1.md)）
- 不要使用全局变量或者共享内存，这会使并发执行的代码变得不安全（参考[14.1小节](14.1.md)）
- `println`函数仅仅是用于调试的目的

最佳实践：对比以下使用方式：  

- 使用正确的方式初始化一个元素是切片的映射，例如`map[type]slice`（参考[8.1.3小节](08.1.md)）
- 一直使用逗号，ok或者checked形式作为类型断言（参考[11.3小节](11.3.md)）
- 使用一个工厂函数创建并初始化自己定义类型（参考[10.2小节](10.2.md)-[18.4小节](18.4.md)）
- 仅当一个结构体的方法想改变结构体时，使用结构体指针作为方法的接受者，否则使用一个结构体值类型[10.6.3小节](10.6.md)

# 误用字符串  
当需要对一个字符串进行频繁的操作时，谨记在go语言中字符串是不可变的（类似java和c#）。使用诸如`a += b`形式连接字符串效率低下，尤其在一个循环内部使用这种形式。这会导致大量的内存开销和拷贝。**应该使用一个字符数组代替字符串，将字符串内容写入一个缓存中。** 例如以下的代码示例：

```go
var b bytes.Buffer
...
for condition {
    b.WriteString(str) // 将字符串str写入缓存buffer
}
    return b.String()
```

注意：由于编译优化和依赖于使用缓存操作的字符串大小，当循环次数大于15时，效率才会更佳。  

# 发生错误时使用defer关闭一个文件  
如果你在一个for循环内部处理一系列文件，你需要使用defer确保文件在处理完毕后被关闭，例如：  

```go
for _, file := range files {
    if f, err = os.Open(file); err != nil {
        return
    }
    // 这是错误的方式，当循环结束时文件没有关闭
    defer f.Close()
    // 对文件进行操作
    f.Process(data)
}
```

但是在循环结尾处的defer没有执行，所以文件一直没有关闭！垃圾回收机制可能会自动关闭文件，但是这会产生一个错误，更好的做法是：  

```go
for _, file := range files {
    if f, err = os.Open(file); err != nil {
        return
    }
    // 对文件进行操作
    f.Process(data)
    // 关闭文件
    f.Close()
 }
```

**defer仅在函数返回时才会执行，在循环的结尾或其他一些有限范围的代码内不会执行。**  

# 何时使用new()和make()  
- 切片、映射和通道，使用make
- 数组、结构体和所有的值类型，使用new 

# 不需要将一个指向切片的指针传递给函数  
切片实际是一个指向潜在数组的指针。我们常常需要把切片作为一个参数传递给函数是因为：实际就是传递一个指向变量的指针，在函数内可以改变这个变量，而不是传递数据的拷贝。  
因此应该这样做：

       func findBiggest( listOfNumbers []int ) int {}

而不是：  

       func findBiggest( listOfNumbers *[]int ) int {}

**当切片作为参数传递时，切记不要解引用切片。**  

# 使用指针指向接口类型  
查看如下程序：`nexter`是一个接口类型，并且定义了一个`next()`方法读取下一字节。函数`nextFew`将`nexter`接口作为参数并读取接下来的`num`个字节，并返回一个切片：这是正确做法。但是`nextFew2`使用一个指向`nexter`接口类型的指针作为参数传递给函数：当使用`next()`函数时，系统会给出一个编译错误：**n.next undefined (type *nexter has no
field or method next)** （译者注：n.next未定义（*nexter类型没有next成员或next方法））  

例 pointer_interface.go (不能通过编译):  

```go
package main
import (
    “fmt”
)
type nexter interface {
    next() byte
}
func nextFew1(n nexter, num int) []byte {
    var b []byte
    for i:=0; i < num; i++ {
        b[i] = n.next()
    }
    return b
}
func nextFew2(n *nexter, num int) []byte {
    var b []byte
    for i:=0; i < num; i++ {
        b[i] = n.next() // 编译错误:n.next未定义（*nexter类型没有next成员或next方法）
    }
    return b
}
func main() {
    fmt.Println(“Hello World!”)
}
```

**永远不要使用一个指针指向一个接口类型，因为它已经是一个指针。**  

# 使用值类型时误用指针  
将一个值类型作为一个参数传递给函数或者作为一个方法的接收者，似乎是对内存的滥用，因为值类型一直是传递拷贝。但是另一方面，值类型的内存是在栈上分配，内存分配快速且开销不大。如果你传递一个指针，而不是一个值类型，go编译器大多数情况下会认为需要创建一个对象，并将对象移动到堆上，所以会导致额外的内存分配：因此当使用指针代替值类型作为参数传递时，我们没有任何收获。  

# 误用协程和通道  
在实际应用中，你不需要并发执行，或者你不需要关注协程和通道的开销，在大多数情况下，通过栈传递参数会更有效率。  

但是，如果你使用`break`、`return`或者`panic`去跳出一个循环，很有可能会导致内存溢出，因为协程正处理某些事情而被阻塞。在实际代码中，通常仅需写一个简单的过程式循环即可。**当且仅当代码中并发执行非常重要，才使用协程和通道。**

# 闭包和协程的使用  
看下面代码：  

```go
package main

import (
    "fmt"
    "time"
)

var values = [5]int{10, 11, 12, 13, 14}

func main() {
    // 版本A:
    for ix := range values { // ix是索引值
        func() {
            fmt.Print(ix, " ")
        }() // 调用闭包打印每个索引值
    }
    fmt.Println()
    // 版本B: 和A版本类似，但是通过调用闭包作为一个协程
    for ix := range values {
        go func() {
            fmt.Print(ix, " ")
        }()
    }
    fmt.Println()
    time.Sleep(5e9)
    // 版本C: 正确的处理方式
    for ix := range values {
        go func(ix interface{}) {
            fmt.Print(ix, " ")
        }(ix)
    }
    fmt.Println()
    time.Sleep(5e9)
    // 版本D: 输出值:
    for ix := range values {
        val := values[ix]
        go func() {
            fmt.Print(val, " ")
        }()
    }
    time.Sleep(1e9)
}

```

输出：

```
0 1 2 3 4

4 4 4 4 4

1 0 3 4 2

10 11 12 13 14
```

版本A调用闭包5次打印每个索引值，版本B也做相同的事，但是通过协程调用每个闭包。按理说这将执行得更快，因为闭包是并发执行的。如果我们阻塞足够多的时间，让所有协程执行完毕，版本B的输出是：`4 4 4 4 4`。为什么会这样？在版本B的循环中，`ix`变量实际是一个单变量，表示每个数组元素的索引值。因为这些闭包都只绑定到一个变量，这是一个比较好的方式，当你运行这段代码时，你将看见每次循环都打印最后一个索引值`4`，而不是每个元素的索引值。因为协程可能在循环结束后还没有开始执行，而此时`ix`值是`4`。  

版本C的循环写法才是正确的：调用每个闭包时将`ix`作为参数传递给闭包。`ix`在每次循环时都被重新赋值，并将每个协程的`ix`放置在栈中，所以当协程最终被执行时，每个索引值对协程都是可用的。注意这里的输出可能是`0 2 1 3 4`或者`0 3 1 2 4`或者其他类似的序列，这主要取决于每个协程何时开始被执行。  

在版本D中，我们输出这个数组的值，为什么版本B不能而版本D可以呢？  

因为版本D中的变量声明是在循环体内部，所以在每次循环时，这些变量相互之间是不共享的，所以这些变量可以单独的被每个闭包使用。  

# 糟糕的错误处理  
## 不要使用布尔值  
像下面代码一样，创建一个布尔型变量用于测试错误条件是多余的：  

```go
var good bool
    // 测试一个错误，`good`被赋为`true`或者`false`
    if !good {
        return errors.New("things aren’t good")
    }
```

立即检测一个错误：  

```go
... err1 := api.Func1()
if err1 != nil { … }
```
## 避免错误检测使代码变得混乱：  
避免写出这样的代码：  

```go
... err1 := api.Func1()
if err1 != nil {
    fmt.Println("err: " + err.Error())
    return
}
err2 := api.Func2()
if err2 != nil {
...
    return
}    
```

首先，包括在一个初始化的`if`语句中对函数的调用。但即使代码中到处都是以`if`语句的形式通知错误（通过打印错误信息）。通过这种方式，很难分辨什么是正常的程序逻辑，什么是错误检测或错误通知。还需注意的是，大部分代码都是致力于错误的检测。通常解决此问题的好办法是尽可能以闭包的形式封装你的错误检测，例如下面的代码：  

```go
func httpRequestHandler(w http.ResponseWriter, req *http.Request) {
    err := func () error {
        if req.Method != "GET" {
            return errors.New("expected GET")
        }
        if input := parseInput(req); input != "command" {
            return errors.New("malformed command")
        }
        // 可以在此进行其他的错误检测
    } ()

        if err != nil {
            w.WriteHeader(400)
            io.WriteString(w, err)
            return
        }
        doSomething() ...
```

这种方法可以很容易分辨出错误检测、错误通知和正常的程序逻辑[更详细...](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/13.5.md)。

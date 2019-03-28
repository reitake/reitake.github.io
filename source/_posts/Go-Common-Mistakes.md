---
title: Go 中常见错误
date: 2019-03-11 21:15:27
tags: [Go, Note]
categories: Go
permalink: Go-Common-Mistakes
---
<center> <font color="#bababa">***Go 语言中的常见错误笔记***</font><br /> </center>
<!--more-->
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

# 不能使用简短声明来设置字段的值  
struct 的变量字段不能使用 := 来赋值以使用预定义的变量来避免解决：  
```go
// 错误示例
type info struct {
    result int
}

func work() (int, error) {
    return 3, nil
}

func main() {
    var data info
    data.result, err := work()  // error: non-name data.result on left side of :=
    fmt.Printf("info: %+v\n", data)
}


// 正确示例
func main() {
    var data info
    var err error   // err 需要预声明

    data.result, err = work()
    if err != nil {
        fmt.Println(err)
        return
    }

    fmt.Printf("info: %+v\n", data)
}
```

# 显式类型的变量无法使用 nil 来初始化  
`nil` 是 interface、function、pointer、map、slice 和 channel 类型变量的默认初始值。但声明时不指定类型，编译器也无法推断出变量的具体类型。  
```go
// 错误示例
func main() {
    var x = nil // error: use of untyped nil
    _ = x
}


// 正确示例
func main() {
    var x interface{} = nil
    _ = x
}
```

# 直接使用值为 nil 的 slice、map  
允许对值为 nil 的 slice 添加元素，但对值为 nil 的 map 添加元素则会造成运行时 panic  
```go
// map 错误示例
func main() {
    var m map[string]int
    m["one"] = 1        // error: panic: assignment to entry in nil map
    // m := make(map[string]int)// map 的正确声明，分配了实际的内存
}    


// slice 正确示例
func main() {
    var s []int
    s = append(s, 1)
}
```

# map 容量  
在创建 map 类型的变量时可以指定容量，但不能像 slice 一样使用 cap() 来检测分配空间的大小：  
```go
// 错误示例
func main() {
    m := make(map[string]int, 99)
    println(cap(m))     // error: invalid argument m1 (type map[string]int) for cap  
}
```

# string 类型的变量值不能为 nil  
不用 `nil` 初始化字符串  
```go
// 错误示例
func main() {
    var s string = nil  // cannot use nil as type string in assignment
    if s == nil {   // invalid operation: s == nil (mismatched types string and nil)
        s = "default"
    }
}


// 正确示例
func main() {
    var s string    // 字符串类型的零值是空串 ""
    if s == "" {
        s = "default"
    }
}
```

# range 遍历 slice 和 array 时混淆了返回值  
Go 中的 `range` 在遍历时会生成 2 个值，第一个是元素索引，第二个是元素的值。  

# slice 和 array 其实是一维数据  
- 使用原始的一维数组：要做好索引检查、溢出检测、以及当数组满时再添加值时要重新做内存分配。  
- 使用“独立”的切片分两步：  
    + 创建外部 slice
    + 对每个内部 slice 进行内存分配。注意内部的 slice 相互独立，使得任一内部 slice 增缩都不会影响到其他的 slice  
```go
// 使用各自独立的 6 个 slice 来创建 [2][3] 的动态多维数组
func main() {
    x := 2
    y := 4
    
    table := make([][]int, x)
    for i  := range table {
        table[i] = make([]int, y)
    }
}
```
- 使用“共享底层数组”的切片:  
    + 创建一个存放原始数据的容器 slice
    + 创建其他的 slice
    + 切割原始 slice 来初始化其他的 slice

# 访问 map 中不存在的 key  
不能通过取出来的值来判断 key 是不是在 map 中。  
检查 key 是否存在可以用 map 直接访问，检查返回的第二个参数即可：  
```go
// 错误的 key 检测方式
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if v := x["two"]; v == "" {
        fmt.Println("key two is no entry")  // 键 two 存不存在都会返回的空字符串
    }
}

// 正确示例
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if _, ok := x["two"]; !ok {
        fmt.Println("key two is no entry")
    }
}
```

# string 类型的值是常量，不可更改  
尝试使用索引遍历字符串，来更新字符串中的个别字符，是不允许的。  
string 类型的值是只读的二进制 byte slice，如果真要修改字符串中的字符，将 string 转为 []byte 修改后，再转为 string 即可：  
```go
// 修改字符串的错误示例
func main() {
    x := "text"
    x[0] = "T"      // error: cannot assign to x[0]
    fmt.Println(x)
}


// 修改示例
func main() {
    x := "text"
    xBytes := []byte(x)
    xBytes[0] = 'T' // 注意此时的 T 是 rune 类型
    x = string(xBytes)
    fmt.Println(x)  // Text
}
```
**注意**： 上边的示例并不是更新字符串的正确姿势，因为一个 UTF8 编码的字符可能会占多个字节，比如汉字就需要 3~4 个字节来存储，此时更新其中的一个字节是错误的。  

更新字串的正确姿势：将 string 转为 rune slice（此时 1 个 rune 可能占多个 byte），直接更新 rune 中的字符  
```go
func main() {
    x := "text"
    xRunes := []rune(x)
    xRunes[0] = '我'
    x = string(xRunes)
    fmt.Println(x)  // 我ext
}
```
# 字符串的长度  
```go
func main() {
    char := "♥"
    fmt.Println(len(char))  // 3
}
```
Go 的内建函数 `len()` 返回的是字符串的 byte 数量，而不是像 Python 中那样是计算 Unicode 字符数。  

如果要得到字符串的字符数，可使用 “unicode/utf8” 包中的 `RuneCountInString(str string) (n int)`
```go
func main() {
    char := "♥"
    fmt.Println(utf8.RuneCountInString(char))   // 1
}
```
**注意**： RuneCountInString 并不总是返回我们看到的字符数，因为有的字符会占用 2 个 rune：  
```go
func main() {
    char := "é"
    fmt.Println(len(char))  // 3
    fmt.Println(utf8.RuneCountInString(char))   // 2
    fmt.Println("cafe\u0301")   // café // 法文的 cafe，实际上是两个 rune 的组合
}
```
# 在多行 array、slice、map 语句中缺少 , 号 
```go
func main() {
    x := []int {
        1,
        2   // syntax error: unexpected newline, expecting comma or }
    }
    y := []int{1,2,}    
    z := []int{1,2} 
    // ...
}
```

# `log.Fatal` 和 `log.Panic` 不只是 log  
log 标准库提供了不同的日志记录等级，与其他语言的日志库不同，Go 的 log 包在调用 Fatal*()、Panic*() 时能做更多日志外的事，如中断程序的执行等：  
```go
func main() {
    log.Fatal("Fatal level log: log entry")     // 输出信息后，程序终止执行
    log.Println("Nomal level log: log entry")
}
```

# 对内建数据结构的操作并不是同步的  
尽管 Go 本身有大量的特性来支持并发，但并不保证并发的数据安全，用户需自己保证变量等数据以原子操作更新。  

goroutine 和 channel 是进行原子操作的好方法，或使用 “sync” 包中的锁。  

# range 迭代 string 得到的值  
range 得到的索引是字符值（Unicode point / rune）第一个字节的位置，与其他编程语言不同，这个索引并不直接是字符在字符串中的位置。  

for range 迭代会尝试将 string 翻译为 UTF8 文本，对任何无效的码点都直接使用 0XFFFD rune（�）UNicode 替代字符来表示。如果 string 中有任何非 UTF8 的数据，应将 string 保存为 byte slice 再进行操作。  

```go
func main() {
    data := "A\xfe\x02\xff\x04"
    for _, v := range data {
        fmt.Printf("%#x ", v)   // 0x41 0xfffd 0x2 0xfffd 0x4   // 错误
    }

    for _, v := range []byte(data) {
        fmt.Printf("%#x ", v)   // 0x41 0xfe 0x2 0xff 0x4   // 正确
    }
}
```

# range 迭代 map  
如果你希望以特定的顺序（如按 key 排序）来迭代 map，要注意每次迭代都可能产生不一样的结果。  

Go 的运行时是有意打乱迭代顺序的，所以你得到的迭代结果可能不一致。但也并不总会打乱，得到连续相同的 5 个迭代结果也是可能的，如：  
```go
func main() {
    m := map[string]int{"one": 1, "two": 2, "three": 3, "four": 4}
    for k, v := range m {
        fmt.Println(k, v)
    }
}
```
# switch 中的 fallthrough 语句  
`switch` 语句中的 `case` 代码块会默认带上 break，不过你可以在 case 代码块末尾使用 fallthrough，强制执行下一个 case 代码块。也可以改写 case 为多条件判断。  

# 自增和自减运算  
多编程语言都自带前置后置的 `++`、`--` 运算。但 Go 特立独行，去掉了前置操作，同时 `++`、`--` 只作为运算符而非表达式。  

# 按位取反  
很多编程语言使用 `~` 作为一元按位取反（NOT）操作符，Go 中用 `^` XOR 操作符来按位取反。  
同时 `^` 也是按位异或（XOR）操作符。  

Go 也有特殊的操作符 AND NOT `&^` 操作符，不同位才取1。

# 运算符的优先级  
优先级列表：  
```go
Precedence    Operator
    5             *  /  %  <<  >>  &  &^
    4             +  -  |  ^
    3             ==  !=  <  <=  >  >=
    2             &&
    1             ||
```

# 不导出的 struct 字段无法被 encode  
以小写字母开头的字段成员是无法被外部直接访问的，所以 `struct` 在进行 json、xml、gob 等格式的 encode 操作时，这些私有字段会被忽略，导出时得到零值：  
```go
func main() {
    in := MyData{1, "two"}
    fmt.Printf("%#v\n", in) // main.MyData{One:1, two:"two"}

    encoded, _ := json.Marshal(in)
    fmt.Println(string(encoded))    // {"One":1}    // 私有字段 two 被忽略了

    var out MyData
    json.Unmarshal(encoded, &out)
    fmt.Printf("%#v\n", out)    // main.MyData{One:1, two:""}
}
```

# 程序退出时还有 goroutine 在执行  
程序默认不等所有 goroutine 都执行完才退出，这点需要特别注意。  

常用解决办法：使用 “WaitGroup” 变量，它会让主程序等待所有 goroutine 执行完毕再退出。  

如果你的 goroutine 要做消息的循环处理等耗时操作，可以向它们发送一条 kill 消息来关闭它们。或直接关闭一个它们都等待接收数据的 channel：  

```go
// 等待所有 goroutine 执行完毕
// 使用传址方式为 WaitGroup 变量传参
// 使用 channel 关闭 goroutine

func main() {
    var wg sync.WaitGroup
    done := make(chan struct{})
    ch := make(chan interface{})

    workerCount := 2
    for i := 0; i < workerCount; i++ {
        wg.Add(1)
        go doIt(i, ch, done, &wg)   // wg 传指针，doIt() 内部会改变 wg 的值
    }

    for i := 0; i < workerCount; i++ {  // 向 ch 中发送数据，关闭 goroutine
        ch <- i
    }

    close(done)
    wg.Wait()
    close(ch)
    fmt.Println("all done!")
}

func doIt(workerID int, ch <-chan interface{}, done <-chan struct{}, wg *sync.WaitGroup) {
    fmt.Printf("[%v] is running\n", workerID)
    defer wg.Done()
    for {
        select {
        case m := <-ch:
            fmt.Printf("[%v] m => %v\n", workerID, m)
        case <-done:
            fmt.Printf("[%v] is done\n", workerID)
            return
        }
    }
}
```

# 若函数 receiver 传参是传值方式，则无法修改参数的原有值  
方法 receiver 的参数与一般函数的参数类似：如果声明为值，那方法体得到的是一份参数的值拷贝，此时对参数的任何修改都不会对原有值产生影响。  

除非 receiver 参数是 map 或 slice 类型的变量，并且是以指针方式更新 map 中的字段、slice 中的元素的，才会更新原有值:  

```go
type data struct {
    num   int
    key   *string
    items map[string]bool
}

func (this *data) pointerFunc() {
    this.num = 7
}

func (this data) valueFunc() {
    this.num = 8
    *this.key = "valueFunc.key"
    this.items["valueFunc"] = true
}

func main() {
    key := "key1"

    d := data{1, &key, make(map[string]bool)}
    fmt.Printf("num=%v  key=%v  items=%v\n", d.num, *d.key, d.items)

    d.pointerFunc() // 修改 num 的值为 7
    fmt.Printf("num=%v  key=%v  items=%v\n", d.num, *d.key, d.items)

    d.valueFunc()   // 修改 key 和 items 的值，num 未修改成功
    fmt.Printf("num=%v  key=%v  items=%v\n", d.num, *d.key, d.items)
}
```

# struct、array、slice 和 map 的值比较  
可以使用相等运算符 == 来比较结构体变量，前提是两个结构体的成员都是可比较的类型。  
```go
type data struct {
    num     int
    fp      float32
    complex complex64
    str     string
    char    rune
    yes     bool
    events  <-chan string
    handler interface{}
    ref     *byte
    raw     [10]byte
}

func main() {
    v1 := data{}
    v2 := data{}
    fmt.Println("v1 == v2: ", v1 == v2) // true
}
```
如果两个结构体中有任意成员是不可比较的，将会造成编译错误。注意数组成员只有在数组元素可比较时候才可比较。  
```go
type data struct {
    num    int
    checks [10]func() bool      // 无法比较
    doIt   func() bool      // 无法比较
    m      map[string]string    // 无法比较
    bytes  []byte           // 无法比较
}

func main() {
    v1 := data{}
    v2 := data{}

    fmt.Println("v1 == v2: ", v1 == v2)
}
```
Go 提供了一些库函数来比较那些无法使用 `==` 比较的变量，比如使用 “reflect” 包的 `DeepEqual()` ：
```go
// 比较相等运算符无法比较的元素
func main() {
    v1 := data{}
    v2 := data{}
    fmt.Println("v1 == v2: ", reflect.DeepEqual(v1, v2))    // true

    m1 := map[string]string{"one": "a", "two": "b"}
    m2 := map[string]string{"two": "b", "one": "a"}
    fmt.Println("v1 == v2: ", reflect.DeepEqual(m1, m2))    // true

    s1 := []int{1, 2, 3}
    s2 := []int{1, 2, 3}
    // 注意两个 slice 相等，值和顺序必须一致
    fmt.Println("v1 == v2: ", reflect.DeepEqual(s1, s2))    // true
}
```
这种比较方式可能比较慢，根据你的程序需求来使用。`DeepEqual()` 还有其他用法：  
```go
func main() {
    var b1 []byte = nil
    b2 := []byte{}
    fmt.Println("b1 == b2: ", reflect.DeepEqual(b1, b2))    // false
}
```
**注意：** `DeepEqual()` 并不总适合于比较 slice

如果要大小写不敏感来比较 byte 或 string 中的英文文本，可以使用 “bytes” 或 “strings” 包的 `ToUpper()` 和 `ToLower()` 函数。比较其他语言的 byte 或 string，应使用 `bytes.EqualFold()` 和 `strings.EqualFold()`  

如果 byte slice 中含有验证用户身份的数据（密文哈希、token 等），不应再使用 `reflect.DeepEqual()`、`bytes.Equal()`、 `bytes.Compare()`。这三个函数容易对程序造成 timing attacks，此时应使用 “crypto/subtle” 包中的 `subtle.ConstantTimeCompare()` 等函数  

# 在 range 迭代 slice、array、map 时通过更新引用来更新元素  
在 range 迭代中，得到的值其实是元素的一份值拷贝，更新拷贝并不会更改原来的元素，即是拷贝的地址并不是原有元素的地址。  

如果要修改原有元素的值，应该使用索引直接访问。  

如果你的集合保存的是指向值的指针，需稍作修改。依旧需要使用索引访问元素，不过可以使用 range 出来的元素直接更新原有值：  

```go
func main() {
    data := []*struct{ num int }{{1}, {2}, {3},}
    for _, v := range data {
        v.num *= 10 // 直接使用指针更新
    }
    fmt.Println(data[0], data[1], data[2])  // &{10} &{20} &{30}
}
```

# slice 中隐藏的数据  
从 slice 中重新切出新 slice 时，新 slice 会引用原 slice 的底层数组。如果跳了这个坑，程序可能会分配大量的临时 slice 来指向原底层数组的部分数据，将导致难以预料的内存使用。  

可以通过拷贝临时 slice 的数据，而不是重新切片来解决：  
```go
func get() (res []byte) {
    raw := make([]byte, 10000)
    fmt.Println(len(raw), cap(raw), &raw[0])    // 10000 10000 0xc420080000
    res = make([]byte, 3)
    copy(res, raw[:3])
    return
}

func main() {
    data := get()
    fmt.Println(len(data), cap(data), &data[0]) // 3 3 0xc4200160b8
}
```

# 旧 slice  
当你从一个已存在的 slice 创建新 slice 时，二者的数据指向相同的底层数组。如果你的程序使用这个特性，那需要注意 “旧”（stale） slice 问题。  

某些情况下，向一个 slice 中追加元素而它指向的底层数组容量不足时，将会重新分配一个新数组来存储数据。而其他 slice 还指向原来的旧底层数组。  
```go
// 超过容量将重新分配数组来拷贝值、重新存储
func main() {
    s1 := []int{1, 2, 3}
    fmt.Println(len(s1), cap(s1), s1)   // 3 3 [1 2 3 ]

    s2 := s1[1:]
    fmt.Println(len(s2), cap(s2), s2)   // 2 2 [2 3]

    for i := range s2 {
        s2[i] += 20
    }
    // 此时的 s1 与 s2 是指向同一个底层数组的
    fmt.Println(s1)     // [1 22 23]
    fmt.Println(s2)     // [22 23]

    s2 = append(s2, 4)  // 向容量为 2 的 s2 中再追加元素，此时将分配新数组来存

    for i := range s2 {
        s2[i] += 10
    }
    fmt.Println(s1)     // [1 22 23]    // 此时的 s1 不再更新，为旧数据
    fmt.Println(s2)     // [32 33 14]
}
```

# 跳出 for-switch 和 for-select 代码块  
没有指定标签的 break 只会跳出 switch/select 语句，若不能使用 return 语句跳出的话，可为 break 跳出标签指定的代码块然后 `goto`。  

# defer 函数的参数值  
对 defer 延迟执行的函数，它的参数会在声明时候就会求出具体值，而不是在执行时才求值：  
```go
// 在 defer 函数中参数会提前求值
func main() {
    var i = 1
    defer fmt.Println("result: ", func() int { return i * 2 }())
    i++
}

// result: 2
```

---
title: Go语言学习笔记
permalink: Golang-Note
date: 2019-03-07 12:26:44
tags: [Go, Note]
categories: Go
---
# 声明    
## 变量
### 单变量声明
```go
    //声明类型，另行赋值
    var v_name v_type
    v_name = dfsf

    //声明值，自动判定类型
    var v_name = value

    //省略var，使用：=
    v_name := value   
```
### 多变量声明  
```go
    //声明同个类型，非全局变量
    var vname1, vname2, vname3 v_type
    vname1, vname2, vname3 = v1, v2, v3

    //声明值，自动判断类型
    var vname1, vname2, vname3 = v1, v2, v3

    //使用:=，只能在函数体中使用
    vname1, vmame2, vname3 := v1, v2, v3

    //一般用于全局变量声明
    var(
        vname1 v_type1
        vname2 v_type2
    )
```
---
## 数组  
`唯一类型，长度固定，已编号`的序列。  
数组长度是数组type的一部分。  
数组最大长度为 2Gb。  
索引超出数组长度时，会 panic。  

### 单维数组声明 
```go
    //指定元素类型及个数
    var arr_name [SIZE] arr_type
    //eg.
    var arr [10] float32

    //数组是值类型，用new()创建
    var arr = new([]int)
```
**初始化数组：**  
```go
    //声明时赋值，{}数据量不能大于[]
    var arr = [5]float32{1.0, 2.2, 4.0, 5.5, 6.0}
    var arr2 = [5]string{3: "abc", 4: "xyz"}

    //[]不设大小，自动判断数组大小
    var arr = [...]float32{1.0, 2.2, 4.0, 5.5, 6.0}

    //声明后单独赋值
    arr[4] = 6.0
```
### 多维数组
```go
    var arr_name [SIZE1][SIZE2]...[SIZEN] arr_type
    //eg.
    var arr3[5][10][4]int
```
**二维数组初始化：**
```go
    a = [3][4]int{  
     {0, 1, 2, 3} ,    /*  第一行索引为 0 */
     {4, 5, 6, 7} ,    /*  第二行索引为 1 */
     {8, 9, 10, 11},   /*  第三行索引为 2 */
    }
    //必须有逗号
```

### 数组操作
* 用 for 遍历：  
```go
    for i:=0; i < len(arr1); i++｛
        arr1[i] = ...
    }
```
* 用 for-range：  
```go
    for i,_:= range arr1 {...}
```
* new() 的区别：
```go
    var arr1 = new([5]int)  // arr1 的类型是 *[5]int
    var arr2 [5]int         // arr2 的类型是 [5]int
```
故传参时，`func1(arr2)` 是数组拷贝，不能修改原数组。要修改，要 `func(&arr2)`
* 把大数组传给函数会消耗很多内存，避免方法：（1）传递数组的指针；（2）使用数组的切片。  

---
## 切片  
是对数组一个连续片段的引用（该数组称为相关数组，通常是匿名的）。  
### 切片声明  
```go
    //声明类型，省略大小
    var slice_name []type

    //用make()创建切片
    var slice_name [] = make([]type, len)
    //也可简写成
    slice_name := make([]type, len, capacity)  //capacity为可选参数

    // 用 new() 创建，与用 make 相同
    new([100]int)[0:50]
    make([]int, 50, 100)
```
new() 和 make() 的区别：  
* new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为*T的内存地址：这种方法**返回一个指向类型为 T，值为 0 的地址的指针**，它适用于值类型如数组和结构体。 
*  make(T) 返回一个类型为 T 的初始值，它只适用于3种内建的引用类型：切片、map 和 channel。  
即 new 函数分配内存，make 函数初始化。  
**切片初始化：**  
```go
    //直接:=声明+初始化
    s := []int{1,2,3}   // cap = len = 3

    //用数组初始化切片
    s := arr[:]     // s := arr[startIndex:endIndex]
    var s []type arr[:]
```
**空(nil)切片：**  
一个切片在未初始化之前默认为 nil，长度为 0。  
```go
    var s []int
    if(s == nil){
        ...
    }
```
### 切片的操作  
* 切片重组（reslice）：切片达到cao上限后可扩容，改变切片长度的过程称为重组reslicing。做法：`slice1 = slice1[0:end]`  
切片扩展 1 位： `sl = sl[0:len(sl)+1]`  
* 切片后移 1 位：`s = s[1:]`,不可前移。
* 注意：不要用指针指向 slice，因为切片本身已经是一个引用类型，所以它本身就是一个指针！  
* for-range 用于切片时，第一个返回值 ix 是索引，第二个返回值 val 是该索引位置的值，val 是值拷贝。
* `copy(sl_to, sl_form)` 切片复制  
* `apend(sl, elem1...)` 切片追加  
* s 是个字符串（本质是字节数组），那可通过 `c := []byte(s)` 来获得字节切片 c。或者 `copy(dst []byte, src string)`    
* 字符串是不可被赋值修改的，要修改可以将字符串转化成字节数字，然后修改数组元素，最后把字节数组转换回字符串：  
```go
    s := "hello"
    c := []byte(s)
    c[0] = 'c'
    s2 := string(c) // s2 == "cello"
```
* `sort` 包实现搜索和排序。搜索元素前必须先排序（因为搜索是二分法）。[sort 官方文档](http://golang.org/pkg/sort/)  
```go
    sort.Ints(arri)     // 升序排序
    func Float64s(a []float64)
    func Stirings(a []string)

    IntsAreSorted(a []int) bool     // 检查是否已被排序
    func SearchInts(a []int, n int) int     // 搜索
```
* append 函数常见操作
```go
    // 切片 b 追加到切片 a 之后：
    a = append(a, b...)
    // 复制切片 a 的元素到新的切片 b 上：
    b = make([]T, len(a))
    copy(b, a)
    // 删除位于索引 i 的元素：
    a = append(a[:i], a[i+1:]...)
    // 切除切片 a 中从索引 i 至 j 位置的元素：
    a = apend(a[:i], a[j:]...)
    // 为切片 a 拓展 j 个元素长度：
    a = append(a, make([]T, j)...)
    // 在索引 i 的位置插入元素 x：
    a = append(a[:i], append([]T{x}, a[i:]...)...)
    // 在索引 i 的位置插入切片 b 的所有元素：
    a = append(a[:i], append(b, a[i:]...)...)
    // 取出位于切片 a 最末尾的元素 x：
    x, a = a[len(a)-1], a[:len(a)-1]
```

### 切片和垃圾回收  
切片的底层是数组，数组实际容量可能大于切片容量，只有没有任何切片指向数组时，底层数组的内存才会被释放，因此可能对导致过多内存被占用。  

---
## 指针
### 指针声明
```go
    var p_name *p_type
    //eg.
    var ip *int
    var fp *float32
```
**指针使用：**
```go
    var a int = 20
    var ip *int
    ip = &a
    fmt.Printf("a 变量的地址是: %x\n", &a)
    fmt.Printf("ip 变量储存的指针地址: %x\n", ip )   /* 指针变量的存储地址 */
    fmt.Printf("*ip 变量的值: %d\n", *ip )          /* 使用指针访问值 */
```
**空指针：**
```go
    var ptr *int
    fmt.Printf("ptr的值为 ： %x\n"), ptr)   /* 空指针的值为零 */
    
    //控指针的判断：
    if(ptr != nil)     /* ptr 不是空指针 */
    if(ptr == nil)    /* ptr 是空指针 */
```
### 数组指针
```go
    package main

import "fmt"

const MAX int = 3

func main() {
   a := []int{10,100,200}
   var i int
   var ptr [MAX]*int;

   for  i = 0; i < MAX; i++ {
      ptr[i] = &a[i] /* 整数地址赋值给指针数组 */
   }

   for  i = 0; i < MAX; i++ {
      fmt.Printf("a[%d] = %d\n", i,*ptr[i] )
   }
}
```
输出结果：
```go
a[0] = 10
a[1] = 100
a[2] = 200
```
---
## 结构体  
一系列相同或不同类型的数据结构构成的集合。
### 定义结构体  
```go
    //定义结构体
    type struct_variable_type struct{
        key1 type;
        key2 type;
        ...
        keyN type;
    }

    //用结构体声明变量
    struct_name := struct_variable_type {value1, value2...,valueN}
    struct_name_name2 := struct_variable_type{key1: value1, key2: value2..., keyN: valueN}

    //访问结构体成员
    结构体.成员名

    //结构体名struct_name可以作为函数参数
```
### 结构体指针  
```go
    //定义指向结构体的指针
    var struct_pointer *struct_variable_type
    struct_pointer = &struct_name

    //使用指针访问成员
    struct_pointer.keyN
```
---
## Map(集合)  
一种无序的键值的集合。可以迭代，不返回顺序。  
通过key来快速检索。  
```go
    //变量声明，不初始化的话，默认map是nil，nil map不能用来存放键值
    var map_variable map[key_data_type]value_data_type
    //或使用make
    map_variable := make(map[key_data_type]value_data_type)

    //eg.
    var countryCapitalMap map[string]string //创建
    countryCapitalMap = make(map[string]string)
    countryCapitalMap["France"] = "Paris" //map插入key - value对
    countryCapitalMap["Italy"] = "罗马"
    countryCapitalMap["Japan"] = "东京"
    countryCapitalMap["India "] = "新德里"
    /* 遍历输出 */
    for country := range countryCapitalMap {
        fmt.Println(country, "首都是", countryCapitalMap[country])
    }
    /* 查看元素在map中是否存在 */
    capital, ok := countryCapitalMap["美国"] //存在ok=true，不存在ok=false
    fmt.Println(capital)
    fmt.Println(ok)
    if ok {
        fmt.Println("美国的首都是", capital)
    } else {
        fmt.Println("美国的首都不存在")
    }
```
### Map的基本操作  
**delete()函数**
删除Map的元素，参数为key。
```go
    delete(countryCapitalMap, "France")
```
---
## 指针
```go
    s := "Hello World!"
    var p *string = &s
```
为了防止内存溢出，Go 语言不允许指针算法（如： `pointer+2`），因此 `c = *p++` 是非法的。  

---
# 常用包  
## `strings` 包  
Go 中使用 `strings` 包对字符串进行操作。  
### 前、后缀  
`HasPrefix` 判断字符串 `s` 是否以 `prefix` 开头：  
```go
    strings.HasPrefix(s, prefix string) bool
```
`HasSuffix` 判断字符串 `s` 是否以 `suffix` 结尾：  
```go
    strings.HasSuffix(s, suffix string) bool
```
### 字符串包含关系  
`Contains` 判断字符串 `s` 是否包含 `substr` :  
```go
    strings.Contains(s, substr string) bool
```
### 索引字符串位置  
`Index` 返回字符串 `str` 在字符串 `s` 中的第一次出现的索引（`str` 的第一个字符的索引），
返回 `-1` 表示 `s` 不包含 `str`：
```go
    strings.Index(s, str string) int
```
`LastIndex` 返回 `str` 在 `s` 中最后出现的所有（第一个字符），  
返回 `-1` 表示 `s` 不包含 `str`：  
```go
    strings.LastIndex(s, str string) int
```
建议用 `IndexRune` 查询非 ASCII 编码的字符在字符串中的位置（返回 `-1` 表示 `s` 不包含 `str`：  ）：  
```go
    strings.IndexRune(s string, r rune) int

    //eg.
    strings.IndexRune("chicken", 99)
    strings.IndexRune("chicken", rune('k'))
```
### 字符串替换  
`Replace` 用于将字符串 `str` 中的前 `n` 个字符串 `old` 替换为字符串 `new`，并范围一个新的字符串，如果 `n = -1` 啧替换所有有 `old` 为 `new`：  
```go
    strings.Replace(str, old, new, n) string
```
### 统计字符串出现的次数
`Count` 用于统计字符串 `str` 中字符串 `s` 出现的非重叠次数：  
```go
    strings.Count(s, str string) int
```
### 重复字符串  
`Repeat` 用于重复 `count` 次字符串 `s` 并返回一个新的字符串：
```go
    strings.Repeat(s, count int) string
```
### 修改字符串大小写  
`ToLower` 讲字符串中的 Unicode 字符全部转换为相应的小写字符：
```go
    strings.Tolower(s) string
```
`ToUpper` 讲字符串中的 Unicode 字符全部转换为相应的大写字符：
```go
    strings.ToUpper(s) string
```
### 修剪字符串  
`strings.TrimSpace(s)` 用于剔除字符串开头和结尾的空白符号；  
`strings.Trim(s, "cut")` 用于剔除字符串开头和结尾的 `cut`；  
`TrimLeft` 或者 `TrimRight` 用于只剔除开头或结尾的字符串。  
### 分割字符串  
`strings.Fields(s)` 会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice。  
`strings.Split(s, sep)` 用于自定义分割符号来对指定字符串进行分割，同样范围 slice。  
因为这 2 个函数都会返回 slice，所以习惯使用 for - range 循环来对其进行处理。  
### 拼接 slice 到字符串  
`Join` 用于将元素类型为 string 的 slice 使用分割符号来拼接组成一个字符串：  
```go
    strings.Join(sl [string], sep string) string
```
### 从字符串中读取内容  
`strings.NewReader(str)` 用于生成一个 'Reader' 并读取字符串中的内容，然后返回指向该 `Reader` 的指针，从其他类型读取内容的函数还有：  
`Read` 从 []byte 中读取内容；  
`ReadByte()` 和 `ReadRune()` 从字符串中读取下一个 byte 或者 rune。  

其他有关字符串操作的文档参考 [官方文档](http://golang.org/pkg/strings/) 或 [国内访问文档](http://docs.studygolang.com/pkg/strings/)。

---
## `Strconv` 包  
与字符串相关的类型转换。  
包含了一些变量用于获取程序运行的操作系统平台下 int 类型所占的位数，如：`strconv.IntSize`。  
任何类型 T 转换为字符串总是成功的。  
数字 → 字符串：
* `strconv.Itoa(i int) string` 返回数字 i 所表示的字符串类型的十进制数。  
* `strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string` 将64位浮点型数字转换为字符串，其中 `fmt` 表示格式（其值可以是 `b`、`e`、`f` 或 `g`），`prec` 表示精度，`bitSize` 则使用 32 表示 float32，用 64 表示 float64。  

将字符串转换为其他类型 tp 并不总是可能的，可能会在运行时抛出错误 `parsing "…": invalid argument`。  
字符串 → 数字类型：  
* `strconv.Atoi(s string) (i int, err error)` 将字符串转换为 int 型。  
* `strconv.ParseFloat(s string, bitSize int) (f float64, err error)` 将字符串转换为 float64 型。  

利用多个返回值的特性（第 1 个是转换后的结果（若成功），第 2 个是可能出现的错误），一般使用如下形式进行从字符串到其它类型的转换：  
```go
    val, err = strconv.Atoi(s)
```
---
## `time` 包：时间和日期  
`time` 包提供了一个数据类型 `time.Time`（作为值使用）以及显示和测量时间和日期的功能函数。  
获取当前时间：`time.Now()`；  
```go
    var t time.Time = time.Now()
    // 或
    t := time.Now()
```
获取时间的一部分： `t.Day`、`t.Minute()`...
自定义时间格式化字符串：eg.`fmt.Printf("%02d.%02d.%4d\n", t.Day(), t.Month(), t.Year())` 将会输出 `08.03.2019`  
Duration 类型表示两个连续时刻所相差的**纳秒数**，类型为 int64。Location 类型映射某个时区的时间，UTC 表示通用协调世界时间。  
包中的一个预定义函数 `func (t Time) Format(layout string) string` 可以根据一个格式化字符串来将一个t转换为相应格式的字符串，你可以使用一些预定义的格式，如：`time.ANSIC` 或 `time.RFC822`。  
```go
    fmt.Println((t.Format("02 Jan 2006 15:04")))
```
如果需要在应用程序在经过一定时间或周期执行某项任务（事件处理的特例），则可以使用 `time.After` 或者 `time.Ticker`。  
另外，`time.Sleep(Duration d)` 可以实现对某个进程（实质上是 goroutine）时长为 d 的暂停。  
其他关于时间操作的文档参考 [官方文档](http://golang.org/pkg/time/) 或 [国内访问页面](http://docs.studygolang.com/pkg/time/)。  

---
# 控制结构  
## if-else 结构  
```go
    // if
    if condition{
        // do something
    }

    // if...else...
    if condition {
        // do something
    } else {
        // do something
    }
```
一些例子：  
1. 判断一个字符串是否为空：
* `if str == "" {...}`
* `if len(str) == 0 {...}`
2. 判断运行 Go 程序的操作系统类型，这可以通过常量 `runtime.GOOS` 来判断。  
```go
    if runtime.GOOS == "windows" {
        ...
    } else {
        ...
    }
```
这段代码一般被放在 init() 函数中执行。 如以下示例演示如何根据操作系统来决定输入结束时的提示：  
```go
    var prompt = "Enter a digit, e.g. 3 "+ "or %s to quit."
    func init() {
        if runtime.GOOS == "windows" {
            prompt = fmt.Sprintf(prompt, "Ctrl+Z, Enter")
        } else {    // Unix-like
            prompt = fmt.Sprintf(prompt, "Ctrl+D")
        }
    }
```
3. 函数 `Abs()` 用于返回一个整型数字的绝对值：  
```go
    func Abs(x int) int {
        if x < 0 {
            return -x
        }
        return x
    }
```
4. `isGrater` 用来比较两个整型数字的大小：  
```go
    func isGreater(x,y int) bool {
        if x > y {
            return true
        }
        return false
    }
```
if 可以包含一个初始化语句（如：给变量赋值），初始化语句后方必须加分号：  
```go
    if val := 10; val > max {
        // do something
    }
```
注意 `:=` 声明的变量作用于仅在 if 结构中。如果变量在 if 结构之前就已经存在，那么在 if 结构中该变量原来的值会被隐藏。  
### comma,ok模式（pattern）  
Go 语言中函数经常用两个返回值来表示是否执行成功：返回某个值以及 true 表示成功；返回零值（或 nil）和 false 表示失败。也可以使用 error 类型的变量来代替第二个返回值：成功执行的话，error 的值为 nil，否则就会包含相应的错误信息。  
**习惯用法**  
```go
    value, err := pack1.Function1(param1)
    if err != nil {
        fmt.Println("An error occured in pack1.Function1 with parameter %v", param1)
        return err
    }
    // 未发生错误，继续执行：
    // 由于本例的函数调用者属于 main 函数，所以程序会直接停止运行。
```
如果要在错误发生的同时终止程序的运行，可以使用 `os` 包的 `Exit` 函数：
```go
    if err != nil {
    fmt.Printf("Program stopping with error %v", err)
    os.Exit(1)
    // （此处的退出代码 1 可以使用外部脚本获取到）
}
```
当没有错误发生时，代码继续运行就是唯一要做的事情，所以 if 语句块后面不需要使用 else 分支。  
## switch 结构
```go
    switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
    }
```
* 测试多个可能符合条件的值，使用逗号分隔，例如： `case val1, val2, val3`。  
* 每个 `case` 分支都是唯一的，从上至下逐一测试，直到匹配为止。  
* 一旦成功匹配某个分支，在执行完相应代码后就会退出整个 switch 代码块，所以不用 `break`。  
* 如果执行完 `case` 后还需要继续执行后续分支的代码，要使用 `fallthrough`。  
* 可以用 `return` 语句来提前结束代码块的执行。（还要时刻确保函数始终有返回值，switch 后再添加相应 `return`）  
* `switch` 后也可不跟变量，`case` 后跟 condition，用起来像链式 if-else。  
* `switch` 后也可跟一个初始化语句，需要加分号 `;`。 
 
## for 结构  
### 计数器形式
基本形式：  
```go
    for 初始化语句; 条件语句; 修饰语句 {}
```
* 特别注意，永远不要在循环体内修改计数器！  

同时使用多个计数器：  
```go
    for i, j := 0, N; i < j; i, j = i+1, j-1 {}
```
### 条件判断的迭代形式  
基本形式：`for 条件语句 {}`  
### 无限循环  
基本形式：`for {}`  
* 需要循环内用 `break` 或 `return` 退出循环体。（`break` 只是退出循环体，`return` 是提前对函数进行返回，不会执行后续代码）  
* 无限循环的经典应用是服务器，用于**不断等待**和**接受新的请求**：
```go
    for t, err = p.Token(); err == nil; t, err = p.Token() {
        ...
    }
```
### for-range 结构  
用于迭代任何一个集合（数组、map）。  
一般形式：`for ix, val := range coll {...}`  
* 需要注意，val始终是集合中索引的值的拷贝，对它修改不会影响集合内原值（除非 `val` 为指针）。  
* 一个字符串是 Unicode 编码的字符（或称之为 `rune`）集合，因此可可以用它迭代字符串：  
```go
    for pos, char := range str {
        ...
    } 
```
## Break 与 continue  
* 一个 break 的作用范围是该语句出现后的最内部的结构。（只会跳出最内层循环）  
* 关键词 continue 忽略剩余循环体直接进入下一次循环过程，执行下次循环前需要判断循环条件。  
* continue 只能用于 for 循环中。  

## 标签与 goto  
* 标签名称是大小写敏感的，一般建议全用大写字母。  
* 配合 goto 跳出多层循环或模拟循环。  
* 不鼓励使用标签和 goto，会导致糟糕的程序设计，可被更可读的方案替代。 
* 一定要用的话，只使用正序的标签（先 goto，后标签），且两者间不能出现新的定义变量语句！  

# 函数  
* Go 中不允许函数重载（function overloading，指可以编写多个同名函数）。  
* 如果需要声明一个在外部定义的函数，只要给出函数名与函数签名，不用给出函数体：  
```go
    func flushICache(begin, end uintptr)    // implemented externally
```
* 函数可以被作为类型来声明使用：  
```go
    type binOp func(int, int) int
```

## 传参和返回  
* 分按值传递（call by value） 按引用传递（call by reference）。  
* 几乎在任何情况下，传递指针的消耗比传递副本来的少。  
* 在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。
* 如果函数要返回四到五个值，可以传递一个切片给函数（如果返回值具有相同类型）或者是传递一个结构体（如果返回值具有不同的类型）。因为传递一个指针允许直接修改变量的值，消耗也更少。

## 传递变长参数  
* 变参函数：函数最有一个参数采用 `...type` 的形式。函数接受某个类型的 slice 的参数。  
```go
    func myFunc(a, b, arg ...int) {}
```
* 解决边长参数不是相同类型的方法：（1）使用结构；（2）使用空接口。  

## defer 和追踪
* 关键词 defer 让我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数。  
* defer 一般用于释放某些已分配的资源。
* 多个 defer 行为被注册，会逆序执行（类似栈），  
* defer 允许我们进行一些函数执行完成后的收尾工作：（1）关闭文件流；（2）解锁一个加锁的资源；（3）打印最终报告；（4）关闭数据库连接。  
* 用 defer 实现代码追踪。
* 用 defer 记录函数的参数与返回值。

## 内置函数  
不需要导入就可使用。
名称 | 说明 |
---- | ----|
close  | 用于管道通信|
len、cap len |用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）|
new、make  |  new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作,new() 是一个函数，不要忘记它的括号|
copy、append |用于复制和连接切片|
panic、recover | 两者均用于错误处理机制|
print、println  | 底层打印函数，在部署环境中建议使用 fmt 包|
complex、real imag  | 用于创建和操作复数|

## 递归函数
经常遇到的问题是栈溢出：大量递归调用导致程序栈内存分配耗尽。可通过一个名为懒惰求值的技术解决，Go 中可以使用管道（channel）和 goroutine来实现。

## 函数作为参数  
eg. 函数 `strings.IndexFunc()`  
其函数签名是 `func IndexFunc(s string, f func(c int) bool) int`，它的返回值是在函数 `f(c)` 返回 true、-1 或从未返回时的索引值。  

## 闭包  
不想给函数起名字时，可以用匿名函数，例如： `func(x, y int) int { return x + y }`。  
匿名函数不能独立存在，要赋值给变量（即保存函数的地址到变量中）：`fplus := func(x, y int) int { return x + y }`,然后用变量名来调用：`fplus(3,4)`，或直接对匿名函数进行调用：`func(x, y int) int { return x + y }(3, 4)`。  
### defer语句和匿名函数
两者经常搭配，可用于改变函数的命名返回值。  
匿名函数还可以配合 `go` 关键字来作为 goroutine 使用。  
匿名函数被称之为闭包：被允许调用定义在起环境下的变量。闭包可使得某个函数捕捉到一些外部状态，如：函数被创建时的状态。或者说：一个闭包继承了函数所声明时的作用域。这种状态（作用域内的变量）都被共享到闭包的环境中，因此这些变量可以在闭包中被操作，直到被销毁。  
闭包经常被用作包装函数：会预先定义好 1 个或多个参数以用于包装。  
另一个应用是用闭包来完成更加简洁的错误检查。  

## 应用闭包：将函数作为返回值  
* 闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。  
* 闭包中使用到的变量可以是在闭包函数体内声明的，也可以是在外部函数声明的。  

## 使用闭包调试  
调试时需要准确知道哪个文件中的具体哪个函数正在执行。可以使用 `runtime` 或 `log` 包中的特殊函数来实现这样的功能。包 runtime 中的函数 `Caller()` 提供了相应的信息，因此可以在需要的时候实现一个 `where()` 闭包函数来打印函数执行的位置：  
```go
    where := func() {
    _, file, line, _ := runtime.Caller(1)
    log.Printf("%s:%d", file, line)
    }
    where()
    // some code
    where()
    // some more code
    where()
```
也可以设置 `log` 包中的 flag 参数来实现：  
```go
    log.SetFlags(log.Llongfile)
    log.Print("")
```
或使用一个更加简短版本的 `where` 函数：  
```go
    var where = log.Print
    func func1() {
    where()
    ... some code
    where()
    ... some code
    where()
}
```

## 计算函数执行时间  
在计算开始签设置一个起始时候，在计算结束时的结束时间，求差值。  
可用 `time` 包中的 `Now()` 和 `Sub` 函数：  
```go
    start := time.Now()
    longCalculation()
    end := time.Now()
    delta := end.Sub(start)
    fmt.Printf("longCalculation took this amount of time: %s\n", delta)
```

## 通过内存缓存来提升性能  
大量计算式，提升性能最有效的一种方式是**避免重复计算**。通过在内存中缓存和重复利用相同计算的结果，成为内存缓存。  
eg. 求斐波那契数列：  
```go
    package main

    import (
        "fmt"
        "time"
    )

    const LIM = 41

    var fibs [LIM]uint64

    func main() {
        var result uint64 = 0
        start := time.Now()
        for i := 0; i < LIM; i++ {
            result = fibonacci(i)
            fmt.Printf("fibonacci(%d) is: %d\n", i, result)
        }
        end := time.Now()
        delta := end.Sub(start)
        fmt.Printf("longCalculation took this amount of time: %s\n", delta)
    }

    func fibonacci(n int) (res uint64) {
        // memoization: check if fibonacci(n) is already known in array:
        if fibs[n] != 0 {
            res = fibs[n]
            return
        }
        if n <= 1 {
            res = 1
        } else {
            res = fibonacci(n-1) + fibonacci(n-2)
        }
        fibs[n] = res
        return
}
```

---
# 常用资料  
## Go 关键字
* 25 个关键字  

break   |default |func   | interface |  select|
-----|-----|-----|-----|-----|
case    |defer |  go|  map |struct|
chan   | else   | goto  |  package |switch|
const  | fallthrough |if|  range|   type|
continue   | for| import|  return|  var|

* 36 个预定义标识符  

append | bool  |  byte  |  cap |close |  complex |complex64 |  complex128|  uint16|
-----|-----|-----|-----|-----|-----|-----|-----|-----|
copy   | false  | float32 |float64| imag|    int |int8   | int16 |  uint32|
int32  | int64  | iota  |  len |make  |  new |nil |panic|   uint64|
print  | println |real |   recover| string | true  |  uint  |  uint8 |  uintptr|









---
*`to be continued...`*
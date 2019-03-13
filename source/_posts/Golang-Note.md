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
    v_name = value

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

组成结构体类型的数据称为**字段（field）**。    

### 定义结构体  
```go
    //定义结构体
    type struct_variable_type struct{
        filed1 type;
        filed2 type;
        ...
        filedN type;
    }

    // 使用 new，会分配内存
    t := new(T)

    // 常用的初始化结构体方式：
    ms := &struct1{10, 15.5, "Chris"}   //是一种简写，底层仍会调用 new()，必须按顺序赋值

    //用结构体声明变量
    struct_name := struct_variable_type {value1, value2...,valueN}
    struct_name_name2 := struct_variable_type{filed1: value1, filed2: value2..., filedN: valueN}

    //访问结构体成员，选择器（selector
    结构体.成员名
    structname.fieldname

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

### 使用工厂方法创建结构体实例  
```go
type File struct {                              //定义了如下的 File 结构体类型
    fd      int     // 文件描述符
    name    string  // 文件名
}

func NewFile(fd int, name string) *File {       // 返回一个指向结构体实例的指针
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}

f := NewFile(10, "./test.txt")                   // 调用工厂方法
```
如果想知道结构体类型T的一个实例占用了多少内存，可以使用：`size := unsafe.Sizeof(T{})`。  

试图 `make()` 一个结构体变量，会引发一个编译错误，这还不是太糟糕，但是 `new()` 一个映射并试图使用数据填充它，将会引发运行时错误！ 因为 `new(Foo)` 返回的是一个指向 `nil` 的指针，它尚未被分配内存。所以在使用 `map` 时要特别谨慎。：  
```go
type Foo map[string]string
type Bar struct {
    thingOne string
    thingTwo int
}

func main() {
    // OK
    y := new(Bar)
    (*y).thingOne = "hello"
    (*y).thingTwo = 1

    // NOT OK
    z := make(Bar) // 编译错误：cannot make type Bar
    (*z).thingOne = "hello"
    (*z).thingTwo = 1

    // OK
    x := make(Foo)
    x["x"] = "goodbye"
    x["y"] = "world"

    // NOT OK
    u := new(Foo)
    (*u)["x"] = "goodbye" // 运行时错误!! panic: assignment to entry in nil map
    (*u)["y"] = "world"
}
```
### 带标签的结构体  

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。标签的内容不可以在一般的编程中使用，只有包 `reflect` 能获取它。 `reflect`包可以在运行时自省类型、属性和方法，比如：在一个变量上调用 `reflect.TypeOf()` 可以获取变量的正确类型，如果变量是一个结构体类型，就可以通过 Field 来索引结构体的字段，然后就可以使用 Tag 属性。  
```go
import (
    "fmt"
    "reflect"
)

type TagType struct { // tags
    field1 bool   "An important answer"
    field2 string "The name of the thing"
    field3 int    "How much there are"
}

func main() {
    tt := TagType{true, "Barak Obama", 1}
    for i := 0; i < 3; i++ {
        refTag(tt, i)
    }
}

func refTag(tt TagType, ix int) {
    ttType := reflect.TypeOf(tt)
    ixField := ttType.Field(ix)
    fmt.Printf("%v\n", ixField.Tag)
}

// 输出：
An important answer
The name of the thing
How much there are
```

### 匿名字段和内嵌结构体  
结构体可以包含一个或多个 **匿名（或内嵌）字段**，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。匿名字段本身可以是一个结构体类型，即 **结构体可以包含内嵌结构体**。  

- 当两个字段命名冲突时候：
    + 外场名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式；  
    + 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正。  

---

## 方法  
Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此方法是一种特殊类型的函数。  

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型。但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：invalid receiver type…。  

最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。  

一个类型加上它的方法等价于面向对象中的一个**类**。一个重要的区别是：在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是：它们必须是同一个包的。  

类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的**方法集**。  

定义方法的一般格式：  
```go
func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }

// 如果方法不需要使用 recv 的值，可以用 _ 替换它：
func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }
```
在方法名之前，func 关键字之后的括号中指定 receiver。  

如果 `recv` 是 receiver 的实例，Method1 是它的方法名，那么方法调用遵循传统的 `object.name` 选择器符号：**recv.Method1()**。  

如果 `recv` 是一个指针，Go 会自动解引用。  

### 函数和方法的区别  
函数将变量作为参数：**Function1(recv)**  

方法在变量上被调用：**recv.Method1()**  

**receiver_type** 叫做 **（接收者）基本类型**，这个类型必须在和方法同样的包中被声明。  

**方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的。**  

### 方法和未导出字段  
[可以建立Set方法来修改未导出的字段](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.6.md#1064-%E6%96%B9%E6%B3%95%E5%92%8C%E6%9C%AA%E5%AF%BC%E5%87%BA%E5%AD%97%E6%AE%B5)  

### 内嵌类型的方法和继承  
将父类型放在子类型中来实现亚型。  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.6.md#1065-%E5%86%85%E5%B5%8C%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%96%B9%E6%B3%95%E5%92%8C%E7%BB%A7%E6%89%BF)  

### 在类型中嵌入功能
主要有两种方法来实现在类型中嵌入功能：  

A：聚合（或组合）：包含一个所需功能类型的具名字段。  

B：内嵌：内嵌（匿名地）所需功能类型。  

[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.6.md#1066-%E5%A6%82%E4%BD%95%E5%9C%A8%E7%B1%BB%E5%9E%8B%E4%B8%AD%E5%B5%8C%E5%85%A5%E5%8A%9F%E8%83%BD)  

### 多重继承  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.6.md#1067-%E5%A4%9A%E9%87%8D%E7%BB%A7%E6%89%BF)  

### 和其他面向对象语言比较 Go 的类型和方法  
在如 C++、Java、C# 和 Ruby 这样的面向对象语言中，方法在类的上下文中被定义和继承：在一个对象上调用方法时，运行时会检测类以及它的超类中是否有此方法的定义，如果没有会导致异常发生。  

在 Go 语言中，这样的继承层次是完全没必要的：如果方法在此类型定义了，就可以调用它，和其他类型上是否存在这个方法没有关系。在这个意义上，Go 具有更大的灵活性。  

Go 不需要一个显式的类定义，如同 Java、C++、C# 等那样，相反地，“类”是通过提供一组作用于一个共同类型的方法集来隐式定义的。类型可以是结构体或者任何用户自定义类型。  

**总结**

在 Go 中，类型就是类（数据和关联的方法）。Go 不知道类似面向对象语言的类继承的概念。继承有两个好处：代码复用和多态。  

在 Go 中，代码复用通过组合和委托实现，多态通过接口的使用来实现：有时这也叫 **组件编程（Component Programming）**。  

许多开发者说相比于类继承，Go 的接口提供了更强大、却更简单的多态行为。  

**备注**

如果真的需要更多面向对象的能力，看一下 [`goop`](https://github.com/losalamos/goop) 包（Go Object-Oriented Programming），它由 Scott Pakin 编写: 它给 Go 提供了 JavaScript 风格的对象（基于原型的对象），并且支持多重继承和类型独立分派，通过它可以实现你喜欢的其他编程语言里的一些结构。  

### 类型的 String() 方法和格式化描述符  
如果类型定义了 `String()` 方法，它会被用在 `fmt.Printf()` 中生成默认的输出：等同于使用格式化描述符 `%v` 产生的输出。还有 `fmt.Print()` 和 `fmt.Println()` 也会自动使用 `String()` 方法。  

[example](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.7.md)  

格式化描述符 `%T` 会给出类型的完全规格，`%#v` 会给出实例的完整输出，包括它的字段（在程序自动生成 `Go` 代码时也很有用）。  

**备注**

不要在 `String()` 方法里面调用涉及 `String()` 方法的方法，它会导致意料之外的错误，比如下面的例子，它导致了一个无限递归调用（`TT.String()` 调用 `fmt.Sprintf`，而 `fmt.Sprintf` 又会反过来调用 `TT.String()`...），很快就会导致内存溢出：

```go
type TT float64

func (t TT) String() string {
    return fmt.Sprintf("%v", t)
}
t.String()
```

---

## Map(集合)  
一种无序的键值的集合。可以迭代，不返回顺序。  
通过key来快速检索。  
声明时不需要知道 map 长度，map 可以动态增长。不存在固定长度或最大限制，但可以选择表明map的初始容量 `capacity`：`make(map[keytype]valuetype, cap)`  
未初始化的 map 的值是 nil。  

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
* 测试键值对  
```go
    _, ok := map1[key1] // 如果key1存在则ok == true，否则ok为false

    if _, ok := map1[key1]; ok {
        // ...
    }
```

* delete()函数
删除Map的元素，参数为key。  
```go
    delete(map1, key1)      // 如果 key1 不存在，该操作不会产生错误。
    delete(countryCapitalMap, "France")
```
* for-range 用法
```go
    for key, value := range map1 {
        ...
    }
```

### 用切片作为 map 的值  
```go
    mp1 := make(map[int][]int)
    mp1 := make(map[int]*[]int)
```

### map 类型的切片  
必须用两次 `make()`，第一次分配切片，第二次分配切片中每个 map 元素：  
```go
    func main() {
        // Version A:
        items := make([]map[int]int, 5)
        for i:= range items {
            items[i] = make(map[int]int, 1)
            items[i][1] = 2
        }
        fmt.Printf("Version A: Value of items: %v\n", items)

        // Version B: NOT GOOD!
        items2 := make([]map[int]int, 5)
        for _, item := range items2 {
            item = make(map[int]int, 1) // item is only a copy of the slice element.
            item[1] = 2 // This 'item' will be lost on the next iteration.
        }
        fmt.Printf("Version B: Value of items: %v\n", items2)
    }

    // 输出结果：
    Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
    Version B: Value of items: [map[] map[] map[] map[] map[]]
```
注意，应当像 A 版本那样通过索引使用切片的 map 元素。在 B 版本中获得的项只是 map 值的一个拷贝而已，所以真正的 map 元素没有得到初始化。  

### map 的排序  
map 默认是无序的。  
如要排序，要将 key（或 value）拷贝到一个切片，在对切片排序（`sort` 包），然后用切片的 for-range 方法打印 key 和 value。  
```go
    var (
        barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
                                "delta": 87, "echo": 56, "foxtrot": 12,
                                "golf": 34, "hotel": 16, "indio": 87,
                                "juliet": 65, "kili": 43, "lima": 98}
    )

    func main() {
        fmt.Println("unsorted:")
        for k, v := range barVal {
            fmt.Printf("Key: %v, Value: %v / ", k, v)
        }
        keys := make([]string, len(barVal))
        i := 0
        for k, _ := range barVal {
            keys[i] = k
            i++
        }
        sort.Strings(keys)
        fmt.Println()
        fmt.Println("sorted:")
        for _, k := range keys {
            fmt.Printf("Key: %v, Value: %v / ", k, barVal[k])
        }
    }
```
如果想要一个排序的列表，最好使用结构体切片，这样更有效：  
```go
    type name struct {
        key string
        value int
    }
```

---
## 指针
```go
    s := "Hello World!"
    var p *string = &s
```
为了防止内存溢出，Go 语言不允许指针算法（如： `pointer+2`），因此 `c = *p++` 是非法的。  

---

## 接口（Interfaces）与反射（reflection）  
接口提供了一种方式来 **说明** 对象的行为：如果谁能搞定这件事，它就可以用在这儿。  
接口定义了一组方法（方法集），但不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。  

定义接口的格式：  
```go
type Namer interface{   // namer 是一个接口类型
    Method1(param_list) return_type
    Method2(param_list) return_type
}
```
按约定，只包含一个方法的接口的名字由方法名加 `[e]r` 后缀组成，如 `Printer`、`Reader`、`Writer`、`Logger`、`Converter` 等。  

类型（如结构体）实现接口方法集中的方法，每个方法的实现说明了次方法是如何作用于该类型的： **即实现接口**，同时方法集也构成了该类型的接口。  

**类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口**。  

**实现某个接口的类型（除了实现接口方法外）可以有其他的方法**。  

**一个类型可以实现多个接口**。  

**接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）**。  

即使接口在类型之后才定义，二者处于不同的包中，被单独编译：只要类型实现了接口中的方法，它就实现了此接口。  

所有这些特性使得接口具有很大的灵活性。  

```go
package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side

}

func main() {
    sq1 := new(Square)
    sq1.side = 5
    var areaIntf Shaper
    areaIntf = sq1      // 赋值给接口类型变量
    // or:
    // areaIntf := Shaper(sq1)
    // or
    // areaIntf := sq1
    fmt.Printf("The square has area: %f\n", areaIntf.Area())
    var a float32 = sq1.Area()
    fmt.Println(a)
}
```
这是 **多态** 的 Go 版本，多态是面向对象编程中一个广为人知的概念：根据当前的类型选择正确的方法，或者说：同一种类型在不同的实例上似乎表现出不同的行为。  
```go
package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

type Rectangle struct {
    length, width float32
}

func (r Rectangle) Area() float32 {
    return r.length * r.width
}

func main() {

    r := Rectangle{5, 3} // Area() of Rectangle needs a value
    q := &Square{5}      // Area() of Square needs a pointer
    // shapes := []Shaper{Shaper(r), Shaper(q)}
    // or shorter
    shapes := []Shaper{r, q}
    fmt.Println("Looping through shapes for area ...")
    for n, _ := range shapes {
        fmt.Println("Shape details: ", shapes[n])
        fmt.Println("Area of this shape is: ", shapes[n].Area())
    }
}
```

### 接口嵌套接口  
一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样：  
```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```
### 类型断言：如何检测和转换接口变量的类型  
一个接口类型的变量 `varI` 中可以包含任何类型的值，必须有一种方式来检测它的 **动态** 类型，即运行时在变量中存储的值的实际类型。通常我们可以使用 **类型断言** 来测试在某个时刻 `varI` 是否包含类型 `T` 的值：  
```go
v := varI.(T)       // unchecked type assertion
```
**varI 必须是一个接口变量**，否则编译器会报错：`invalid type assertion: varI.(T) (non-interface type (type of varI) on left)` 。  

类型断言可能是无效的，更安全的方式是使用以下形式来进行类型断言：  
```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```
如果转换合法，`v` 是 `varI` 转换到类型 `T` 的值，`ok` 会是 `true`；否则 `v` 是类型 `T` 的零值，`ok` 是 `false`，也没有运行时错误发生。  

**应该总是使用上面的方式来进行类型断言**。  
```go
package main

import (
    "fmt"
    "math"
)

type Square struct {
    side float32
}

type Circle struct {
    radius float32
}

type Shaper interface {
    Area() float32
}

func main() {
    var areaIntf Shaper
    sq1 := new(Square)
    sq1.side = 5

    areaIntf = sq1
    // Is Square the type of areaIntf?
    if t, ok := areaIntf.(*Square); ok {
        fmt.Printf("The type of areaIntf is: %T\n", t)
    }
    if u, ok := areaIntf.(*Circle); ok {
        fmt.Printf("The type of areaIntf is: %T\n", u)
    } else {
        fmt.Println("areaIntf does not contain a variable of type Circle")
    }
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

func (ci *Circle) Area() float32 {
    return ci.radius * ci.radius * math.Pi
}
```
### 类型判断：type-switch  
接口变量的类型也可以使用一种特殊形式的 `switch` 来检测：**type-switch** ：  
```go
switch t := areaIntf.(type) {
case *Square:
    fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
    fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
    fmt.Printf("nil value: nothing to check?\n")
default:
    fmt.Printf("Unexpected type %T\n", t)
}
```
输出：  
```go
Type Square *main.Square with value &{5}
```
变量 `t` 得到了 `areaIntf` 的值和类型， 所有 `case` 语句中列举的类型（`nil` 除外）都必须实现对应的接口（在上例中即 `Shaper`），如果被检测类型没有在 `case` 语句列举的类型中，就会执行 `default` 语句。  

可以用 `type-switch` 进行运行时类型分析，但是在 `type-switch` 不允许有 `fallthrough` 。  

如果仅仅是测试变量的类型，不用它的值，那么就可以不需要赋值语句，比如：  

```go
switch areaIntf.(type) {
case *Square:
    // TODO
case *Circle:
    // TODO
...
default:
    // TODO
}
```

下面的代码片段展示了一个类型分类函数，它有一个可变长度参数，可以是任意类型的数组，它会根据数组元素的实际类型执行不同的动作：  

```go
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
            fmt.Printf("Param #%d is a bool\n", i)
        case float64:
            fmt.Printf("Param #%d is a float64\n", i)
        case int, int64:
            fmt.Printf("Param #%d is a int\n", i)
        case nil:
            fmt.Printf("Param #%d is a nil\n", i)
        case string:
            fmt.Printf("Param #%d is a string\n", i)
        default:
            fmt.Printf("Param #%d is unknown\n", i)
        }
    }
}
```
可以这样调用此方法：`classifier(13, -14.3, "BELGIUM", complex(1, 2), nil, false)` 。  

在处理来自于外部的、类型未知的数据时，比如解析诸如 JSON 或 XML 编码的数据，类型测试和转换会非常有用。  

### 测试一个值是否实现了某个接口  
假定 `v` 是一个值，然后我们想测试它是否实现了 `Stringer` 接口，可以这样做：

```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```
接口是一种契约，实现类型必须满足它，它描述了类型的行为，规定类型可以做什么。接口彻底将类型能做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是**多态的本质**。  

编写参数是接口变量的函数，这使得它们更具有一般性。  

**使用接口使代码更具有普适性。**  

### 使用方法集与接口  

作用于变量上的方法实际上是不区分变量到底是指针还是值的。当碰到接口类型值时，这会变得有点复杂，原因是接口变量中存储的具体值是不可寻址的，幸运的是，如果使用不当编译器会给出错误。  

**总结**  

在接口上调用方法时，必须有和方法定义时相同的接收者类型或者是可以从具体类型 `P` 直接可以辨识的：  

- 指针方法可以通过指针调用
- 值方法可以通过值调用
- 接收者是值的方法可以通过指针调用，因为指针会首先被解引用
- 接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址

将一个值赋值给一个接口时，编译器会确保所有可能的接口方法都可以在此值上被调用，因此不正确的赋值在编译期就会失败。  

**译注**  

Go 语言规范定义了接口方法集的调用规则：  

- 类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集
- 类型 T 的可调用方法集包含接受者为 T 的所有方法
- 类型 T 的可调用方法集不包含接受者为 *T 的方法  

### 第一个例子：使用 Sorter 接口排序  
[The way to go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.7.md)  

### 第二个例子：读和写
`io` 包提供了用于读和写的接口 `io.Reader` 和 `io.Writer`：  
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```
一个对象要是可读的，它必须实现 `io.Reader` 接口，这个接口只有一个签名是 `Read(p []byte) (n int, err error)` 的方法，它从调用它的对象上读取数据，并把读到的数据放入参数中的字节切片中，然后返回读取的字节数和一个 `error` 对象，如果没有错误发生返回 `nil`，如果已经到达输入的尾端，会返回 `io.EOF("EOF")`，如果读取的过程中发生了错误，就会返回具体的错误信息。类似地，一个对象要是可写的，它必须实现 `io.Writer` 接口，这个接口也只有一个签名是 `Write(p []byte) (n int, err error)` 的方法，它将指定字节切片中的数据写入调用它的对象里，然后返回实际写入的字节数和一个 `error` 对象（如果没有错误发生就是 `nil`）。  

`io` 包里的 `Readers` 和 `Writers` 都是不带缓冲的，`bufio` 包里提供了对应的带缓冲的操作，在读写 `UTF-8` 编码的文本文件时它们尤其有用。  

[The way to go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.8.md)  

### 空接口  
**空接口或者最小接口** 不包含任何方法，它对实现不做任何要求：  
```go
type Any interface {}
```
任何其他类型都实现了空接口，`any` 或 `Any` 是空接口一个很好的别名或缩写。  

可以给一个空接口类型的变量 `var val interface {}` 赋任何类型的值。  
```go
package main
import "fmt"

var i = 5
var str = "ABC"

type Person struct {
    name string
    age  int
}

type Any interface{}

func main() {
    var val Any
    val = 5
    fmt.Printf("val has the value: %v\n", val)
    val = str
    fmt.Printf("val has the value: %v\n", val)
    pers1 := new(Person)
    pers1.name = "Rob Pike"
    pers1.age = 55
    val = pers1
    fmt.Printf("val has the value: %v\n", val)
    switch t := val.(type) {
    case int:
        fmt.Printf("Type int %T\n", t)
    case string:
        fmt.Printf("Type string %T\n", t)
    case bool:
        fmt.Printf("Type boolean %T\n", t)
    case *Person:
        fmt.Printf("Type pointer to Person %T\n", t)
    default:
        fmt.Printf("Unexpected type %T", t)
    }
}

// 输出：
val has the value: 5
val has the value: ABC
val has the value: &{Rob Pike 55}
Type pointer to Person *main.Person
```
空接口在 `type-switch` 中联合 `lambda` 函数的用法：  
```go
package main

import "fmt"

type specialString string

var whatIsThis specialString = "hello"

func TypeSwitch() {
    testFunc := func(any interface{}) {
        switch v := any.(type) {
        case bool:
            fmt.Printf("any %v is a bool type", v)
        case int:
            fmt.Printf("any %v is an int type", v)
        case float32:
            fmt.Printf("any %v is a float32 type", v)
        case string:
            fmt.Printf("any %v is a string type", v)
        case specialString:
            fmt.Printf("any %v is a special String!", v)
        default:
            fmt.Println("unknown type!")
        }
    }
    testFunc(whatIsThis)
}

func main() {
    TypeSwitch()
}

// 输出：
any hello is a special String!
```
### 构建通用类型或包含不同类型变量的数组  
通过使用空接口。让我们给空接口定一个别名类型 `Element`：`type Element interface{}`  
然后定义一个容器类型的结构体 `Vector`，它包含一个 `Element` 类型元素的切片：  
```go
type Vector struct {
    a []Element
}
```
`Vector` 里能放任何类型的变量，因为任何类型都实现了空接口，实际上 `Vector` 里放的每个元素可以是不同类型的变量。我们为它定义一个 `At()` 方法用于返回第 `i` 个元素：  
```go
func (p *Vector) At(i int) Element {
    return p.a[i]
}
```
再定一个 `Set()` 方法用于设置第 `i` 个元素的值：
```go
func (p *Vector) Set(i int, e Element) {
    p.a[i] = e
}
```
`Vector` 中存储的所有元素都是 `Element` 类型，要得到它们的原始类型（unboxing：拆箱）需要用到类型断言。  

### 复制数据切片至空接口切片  
假设你有一个 `myType` 类型的数据切片，你想将切片中的数据复制到一个空接口切片中，必须使用 `for-range` 语句来一个一个显式地复制：  
```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
    interfaceSlice[i] = d
}
```

### 通用类型的节点数据结构  
[The way to Go 参考内容]（https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.9.md#1194-%E9%80%9A%E7%94%A8%E7%B1%BB%E5%9E%8B%E7%9A%84%E8%8A%82%E7%82%B9%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84）  

### 接口到接口  
[The way to Go 参考内容]（https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.9.md#1195-%E6%8E%A5%E5%8F%A3%E5%88%B0%E6%8E%A5%E5%8F%A3）  

### 反射包
反射是用程序检查其所拥有的结构，尤其是类型的一种能力；这是元编程的一种形式。反射可以在运行时检查类型和变量，例如它的大小、方法和 `动态`
的调用这些方法。这对于没有源代码的包尤其有用。这是一个强大的工具，除非真得有必要，否则应当避免使用或小心使用。  

变量的最基本信息就是类型和值：反射包的 `Type` 用来表示一个 Go 类型，反射包的 `Value` 为 Go 值提供了反射接口。  

[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.10.md)  

### Printf 和反射  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.11.md)  

### 接口与动态类型  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.12.md)  

### 总结：Go 中的面向对象  

Go 没有类，而是松耦合的类型、方法对接口的实现。  

OO 语言最重要的三个方面分别是：封装，继承和多态，在 Go 中它们是怎样表现的呢？  

- 封装（数据隐藏）：和别的 OO 语言有 4 个或更多的访问层次相比，Go 把它简化为了 2 层（参见 4.2 节的可见性规则）:  

    1）包范围内的：通过标识符首字母小写，`对象` 只在它所在的包内可见  

    2）可导出的：通过标识符首字母大写，`对象` 对所在包以外也可见  

类型只拥有自己所在包中定义的方法。  

- 继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现  
- 多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。  


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

# 包（package）  
## 标准库  
内置在 Go 语言中的，150 个以上。  
- `unsafe`: 包含了一些打破 Go 语言“类型安全”的命令，一般的程序中不会被使用，可用在 C/C++ 程序的调用中。
- `syscall`-`os`-`os/exec`:  
    - `os`: 提供给我们一个平台无关性的操作系统功能接口，采用类Unix设计，隐藏了不同操作系统间差异，让不同的文件系统和操作系统对象表现一致。  
    - `os/exec`: 提供我们运行外部操作系统命令和程序的方式。  
    - `syscall`: 底层的外部包，提供了操作系统底层调用的基本接口。
- `archive/tar` 和 `/zip-compress`：压缩(解压缩)文件功能。
- `fmt`-`io`-`bufio`-`path/filepath`-`flag`:  
    - `fmt`: 提供了格式化输入输出功能。  
    - `io`: 提供了基本输入输出功能，大多数是围绕系统功能的封装。  
    - `bufio`: 缓冲输入输出功能的封装。  
    - `path/filepath`: 用来操作在当前系统中的目标文件名路径。  
    - `flag`: 对命令行参数的操作。　　
- `strings`-`strconv`-`unicode`-`regexp`-`bytes`:  
    - `strings`: 提供对字符串的操作。  
    - `strconv`: 提供将字符串转换为基础类型的功能。
    - `unicode`: 为 unicode 型的字符串提供特殊的功能。
    - `regexp`: 正则表达式功能。  
    - `bytes`: 提供对字符型分片的操作。  
    - `index/suffixarray`: 子字符串快速查询。
- `math`-`math/cmath`-`math/big`-`math/rand`-`sort`:  
    - `math`: 基本的数学函数。  
    - `math/cmath`: 对复数的操作。  
    - `math/rand`: 伪随机数生成。  
    - `sort`: 为数组排序和自定义集合。  
    - `math/big`: 大数的实现和计算。  　　
- `container`-`/list-ring-heap`: 实现对集合的操作。  
    - `list`: 双链表。
    - `ring`: 环形链表。
- `time`-`log`:  
    - `time`: 日期和时间的基本操作。  
    - `log`: 记录程序运行时产生的日志,我们将在后面的章节使用它。
- `encoding/json`-`encoding/xml`-`text/template`:
    - `encoding/json`: 读取并解码和写入并编码 JSON 数据。  
    - `encoding/xml`:简单的 XML1.0 解析器。  
    - `text/template`:生成像 HTML 一样的数据与文本混合的数据驱动模板。  
- `net`-`net/http`-`html`
    - `net`: 网络数据的基本操作。  
    - `http`: 提供了一个可扩展的 HTTP 服务器和基础客户端，解析 HTTP 请求和回复。  
    - `html`: HTML5 解析器。  
- `runtime`: Go 程序运行时的交互操作，例如垃圾回收和协程创建。  
- `reflect`: 实现通过程序运行时反射，让程序操作任意类型的变量。  

`exp` 包中有许多将被编译为新包的实验性的包。它们将成为独立的包在下次稳定版本发布的时候。如果前一个版本已经存在了，它们将被作为过时的包被回收。然而 Go1.0 发布的时候并不包含过时或者实验性的包。

## regexp 包  
正则表达式语法和使用见 [维基百科](http://en.wikipedia.org/wiki/Regular_expression)  
简单模式，用 `Match` 方法即可：
```go
    ok, _ := regexp.Match(pat, []byte(searchIn))        // ok 将返回 true 或 false
```
也可以用 `MatchString`：
```go
    ok, _ := regexp.MatchString(pat, searchIn)  
```
更多方法中，必须先将正则通过 `Compile` 方法返回一个 Regexp 对象。  
`Compile` 函数也可能返回一个错误，我们在使用时忽略对错误的判断是因为我们确信自己正则表达式是有效的。当用户输入或从数据中获取正则表达式的时候，我们有必要去检验它的正确性。另外我们也可以使用 `MustCompile` 方法，它可以像 `Compile` 方法一样检验正则的有效性，但是当正则不合法时程序将 panic。  

## 锁和 sync 包  
多个线程同时操作一个变量时，可能会出错，需要锁。 这种锁的机制是通过 sync 包中 Mutex 来实现的。sync 来源于 "synchronized" 一词，这意味着线程将有序的对同一变量进行访问。  
`sync.Mutex` 是一个互斥锁，它的作用是守护在临界区入口来确保同一时间只能有一个线程进入临界区。  
```go
import  "sync"

type Info struct {      // 假设 info 是一个需要上锁的放在共享内存中的变量
    mu sync.Mutex
    // ... other fields, e.g.: Str string
}

// 如果一个函数要改变这个变量：
func Update(info *Info) {
    info.mu.Lock()
    // critical section:
    info.Str = // new value
    // end critical section
    info.mu.Unlock()
}
```
eg. 实现一个可以上锁的共享缓冲器:  
```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```
在 sync 包中还有一个 `RWMutex` 锁：他能通过 `RLock()` 来允许同一时间多个线程对变量进行读操作，但是只能一个线程进行写操作。如果使用 `Lock()` 将和普通的 `Mutex` 作用相同。包中还有一个方便的 `Once` 类型变量的方法 `once.Do(call)`，这个方法确保被调用函数只能被调用一次。  

相对简单的情况下，通过使用 sync 包可以解决同一时间只能一个线程访问变量或 map 类型数据的问题。如果这种方式导致程序明显变慢或者引起其他问题，我们要重新思考来通过 goroutines 和 channels 来解决问题，这是在 Go 语言中所提倡用来实现并发的技术。我们将在第 14 章对其深入了解，并在第 14.7 节中对这两种方式进行比较。

## 精密计算和 big 包  
对于整数的高精度计算 Go 语言中提供了 big 包。其中包含了 math 包：有用来表示大整数的 `big.Int` 和表示大有理数的 `big.Rat` 类型（可以表示为 2/5 或 3.1416 这样的分数，而不是无理数或 π）。这些类型可以实现任意位类型的数字，只要内存足够大。缺点是更大的内存和处理开销使它们使用起来要比内置的数字类型慢很多。

大的整型数字是通过 big.NewInt(n) 来构造的，其中 n 为 int64 类型整数。而大有理数是用过 big.NewRat(N,D) 方法构造。N（分子）和 D（分母）都是 int64 型整数。因为 Go 语言不支持运算符重载，所以所有大数字类型都有像是 Add() 和 Mul() 这样的方法。它们作用于作为 receiver 的整数和有理数，大多数情况下它们修改 receiver 并以 receiver 作为返回结果。因为没有必要创建 big.Int 类型的临时变量来存放中间结果，所以这样的运算可通过内存链式存储。

## 自定义包和可见性  
当写自己包的时候，要使用短小的不含有 `_`(下划线)的小写单词来为文件命名。  

import 的一般格式如下：
```go
import "包的路径或 URL 地址"   // 路径指当前目录的相对路径
// eg.
import "./pack1/pack1"
import "github.com/org1/pack1”
```

Import with _ :pack1包只导入其副作用，也就是说，只执行它的init函数并初始化其中的全局变量。  
```go
import _ "./pack1/pack1"
```

导入外部安装包：  
```go
go install codesite.ext/author/goExample/goex   // ←网址
```
将一个名为 `codesite.ext/author/goExample/goex` 的 map 安装在 `$GOROOT/src/` 目录下。  

在 `http://golang.org/cmd/goinstall/` 的 `go install` 文档中列出了一些广泛被使用的托管在网络代码仓库的包的导入路径  

**包的初始化:**

程序的执行开始于导入包，初始化 main 包然后调用 main 函数。  

一个没有导入的包将通过分配初始值给所有的包级变量和调用源码中定义的包级 init 函数来初始化。一个包可能有多个 init 函数甚至在一个源码文件中。它们的执行是无序的。这是最好的例子来测定包的值是否只依赖于相同包下的其他值或者函数。  

init 函数是不能被调用的。  

导入的包在包自身初始化前被初始化，而一个包在程序执行中只能初始化一次。  


## 为自定义包使用 godoc
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.6.md)  

## 使用 go install 安装自定义包  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.7.md)  

## 自定义包的目录结构、go install 和 go test  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.8.md)  

## 通过 Git 打包和安装  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.9.md)  

## Go 的外部包和项目  
着手自己 Go 项目前，最好查下是否有存在的第三方包或项目可使用。大多可通过 go install 安装。  

[Go Walker](https://gowalker.org) 支持查询。  

目前已经有许多非常好的外部库，如：

- MySQL(GoMySQL), PostgreSQL(go-pgsql), MongoDB (mgo, gomongo), CouchDB (couch-go), ODBC (godbcl), Redis (redis.go) and SQLite3 (gosqlite) database drivers
- SDL bindings
- Google's Protocal Buffers(goprotobuf)
- XML-RPC(go-xmlrpc)
- Twitter(twitterstream)
- OAuth libraries(GoAuth)

## 在 Go 程序中使用外部库  
[The way to Go 参考内容](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.11.md)  

---

# 读写数据  
## 读取用户的输入  
读取用户的键盘（控制台）输入，从键盘和标准输入 `os.Stdin` 读取输入，最简单的办法是用 `fmt` 包提供的 Scan 和 Sscan 开头的函数：  
```go
package main

import "fmt"

var (
    firstName, lastName, s string
    i                      int
    f                      float32
    input                  = "56.12 / 5212 / Go"
    format                 = "%f / %d / %s"
)

func main() {
    fmt.Println("Please enter your full name:")
    fmt.Scanln(&firstName, &lastName)
    // fmt.Scanf("%s %s", &firstName, &lastName)
    fmt.Printf("Hi %s %s\n", firstName, lastName)
    fmt.Sscanf(input, format, &f, &i, &s)
    fmt.Println("Frome the string we read:", f, i, s)
    // 输出结果: From the string we read: 56.12 5212 Go
}
```
`Scanln` 扫描来自标准输入的文本，将空格分隔的值依次存放到后续的参数内，直到碰到换行。`Scanf` 与其类似，除了 `Scanf` 的第一个参数用作格式字符串，用来决定如何读取。`Sscan` 和以 `Sscan` 开头的函数则是从字符串读取，除此之外，与 `Scanf` 相同。如果这些函数读取到的结果与您预想的不同，您可以检查成功读入数据的个数和返回的错误。  

也可以用 `bufio` 包提供的缓冲读取（buffered reader）来读取数据：  
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

var inputReader *bufio.Reader
var input string
var err error

func main() {
    inputReader = bufio.NewReader(os.Stdin)
    fmt.Println("Please enter some input:")
    input, err = inputReader.ReadString('\n')
    if err == nil {
        fmt.Printf("The input was: %s\n", input)
    }
}
```
`inputReader` 是一个指向 `bufio.Reader` 的指针。`inputReader := bufio.NewReader(os.Stdin)` 这行代码，将会创建一个读取器，并将其与标准输入绑定。  

`bufio.NewReader()` 构造函数的签名为：`func NewReader(rd io.Reader) *Reader`  

该函数的实参可以是满足 `io.Reader` 接口的任意对象（任意包含有适当的 `Read()` 方法的对象，请参考[章节11.8](11.8.md)），函数返回一个新的带缓冲的 `io.Reader` 对象，它将从指定读取器（例如 `os.Stdin`）读取内容。  

返回的读取器对象提供一个方法 `ReadString(delim byte)`，该方法从输入中读取内容，直到碰到 `delim` 指定的字符，然后将读取到的内容连同 `delim` 字符一起放到缓冲区。  

`ReadString` 返回读取到的字符串，如果碰到错误则返回 `nil`。如果它一直读到文件结束，则返回读取到的字符串和 `io.EOF`。如果读取过程中没有碰到 `delim` 字符，将返回错误 `err != nil`。  

在上面的例子中，我们会读取键盘输入，直到回车键（\n）被按下。  

屏幕是标准输出 `os.Stdout`；`os.Stderr` 用于显示错误信息，大多数情况下等同于 `os.Stdout`。  

一般情况下，我们会省略变量声明，而使用 `:=`，例如：  
```go
inputReader := bufio.NewReader(os.Stdin)
input, err := inputReader.ReadString('\n')
```
第二个例子从键盘读取输入，使用了 `switch` 语句：  
```go
package main
import (
    "fmt"
    "os"
    "bufio"
)

func main() {
    inputReader := bufio.NewReader(os.Stdin)
    fmt.Println("Please enter your name:")
    input, err := inputReader.ReadString('\n')

    if err != nil {
        fmt.Println("There were errors reading, exiting program.")
        return
    }

    fmt.Printf("Your name is %s", input)
    // For Unix: test with delimiter "\n", for Windows: test with "\r\n"
    switch input {
    case "Philip\r\n":  fmt.Println("Welcome Philip!")
    case "Chris\r\n":   fmt.Println("Welcome Chris!")
    case "Ivo\r\n":     fmt.Println("Welcome Ivo!")
    default: fmt.Printf("You are not welcome here! Goodbye!")
    }

    // version 2:   
    switch input {
    case "Philip\r\n":  fallthrough
    case "Ivo\r\n":     fallthrough
    case "Chris\r\n":   fmt.Printf("Welcome %s\n", input)
    default: fmt.Printf("You are not welcome here! Goodbye!\n")
    }

    // version 3:
    switch input {
    case "Philip\r\n", "Ivo\r\n":   fmt.Printf("Welcome %s\n", input)
    default: fmt.Printf("You are not welcome here! Goodbye!\n")
    }
}
```
## 文件读写  
### 读文件  
在 Go 语言中，文件使用指向 `os.File` 类型的指针来表示的，也叫做文件句柄。我们在前面章节使用到过标准输入 `os.Stdin` 和标准输出 `os.Stdout`，他们的类型都是 `*os.File`：  
```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
)

func main() {
    inputFile, inputError := os.Open("input.dat")
    if inputError != nil {
        fmt.Printf("An error occurred on opening the inputfile\n" +
            "Does the file exist?\n" +
            "Have you got acces to it?\n")
        return // exit the function on error
    }
    defer inputFile.Close()

    inputReader := bufio.NewReader(inputFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        fmt.Printf("The input was: %s", inputString)
        if readerError == io.EOF {
            return
        }      
    }
}
```
变量 `inputFile` 是 `*os.File` 类型的。该类型是一个结构，表示一个打开文件的描述符（文件句柄）。然后，使用 `os` 包里的 `Open` 函数来打开一个文件。该函数的参数是文件名，类型为 `string`。在上面的程序中，我们以只读模式打开 `input.dat` 文件。  

如果文件不存在或者程序没有足够的权限打开这个文件，Open函数会返回一个错误：`inputFile, inputError = os.Open("input.dat")`。如果文件打开正常，我们就使用 `defer inputFile.Close()` 语句确保在程序退出前关闭该文件。然后，我们使用 `bufio.NewReader` 来获得一个读取器变量。  

通过使用 `bufio` 包提供的读取器（写入器也类似），如上面程序所示，我们可以很方便的操作相对高层的 string 对象，而避免了去操作比较底层的字节。  

接着，我们在一个无限循环中使用 `ReadString('\n')` 或 `ReadBytes('\n')` 将文件的内容逐行（行结束符 '\n'）读取出来。  

**注意：** 在之前的例子中，我们看到，Unix和Linux的行结束符是 \n，而Windows的行结束符是 \r\n。在使用 `ReadString` 和 `ReadBytes` 方法的时候，我们不需要关心操作系统的类型，直接使用 \n 就可以了。另外，我们也可以使用 `ReadLine()` 方法来实现相同的功能。  

一旦读取到文件末尾，变量 `readerError` 的值将变成非空（事实上，常量 `io.EOF` 的值是 true），我们就会执行 `return` 语句从而退出循环。  

**其他类似函数：**  

**1) 将整个文件的内容读到一个字符串里：**  

如果您想这么做，可以使用 `io/ioutil` 包里的 `ioutil.ReadFile()` 方法，该方法第一个返回值的类型是 `[]byte`，里面存放读取到的内容，第二个返回值是错误，如果没有错误发生，第二个返回值为 nil。请看示例 12.5。类似的，函数 `WriteFile()` 可以将 `[]byte` 的值写入文件。  

```go
package main
import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    inputFile := "products.txt"
    outputFile := "products_copy.txt"
    buf, err := ioutil.ReadFile(inputFile)
    if err != nil {
        fmt.Fprintf(os.Stderr, "File Error: %s\n", err)
        // panic(err.Error())
    }
    fmt.Printf("%s\n", string(buf))
    err = ioutil.WriteFile(outputFile, buf, 0644) // oct, not hex
    if err != nil {
        panic(err.Error())
    }
}
```
**2) 带缓冲的读取**  

在很多情况下，文件的内容是不按行划分的，或者干脆就是一个二进制文件。在这种情况下，`ReadString()`就无法使用了，我们可以使用 `bufio.Reader` 的 `Read()`，它只接收一个参数：  

```go
buf := make([]byte, 1024)
...
n, err := inputReader.Read(buf)
if (n == 0) { break}
```

变量 n 的值表示读取到的字节数.  

**3) 按列读取文件中的数据**  

如果数据是按列排列并用空格分隔的，你可以使用 `fmt` 包提供的以 FScan 开头的一系列函数来读取他们。请看以下程序，我们将 3 列的数据分别读入变量 v1、v2 和 v3 内，然后分别把他们添加到切片的尾部。  

```go
package main
import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Open("products2.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    var col1, col2, col3 []string
    for {
        var v1, v2, v3 string
        _, err := fmt.Fscanln(file, &v1, &v2, &v3)
        // scans until newline
        if err != nil {
            break
        }
        col1 = append(col1, v1)
        col2 = append(col2, v2)
        col3 = append(col3, v3)
    }

    fmt.Println(col1)
    fmt.Println(col2)
    fmt.Println(col3)
}
```

输出结果：  

```
[ABC FUNC GO]
[40 56 45]
[150 280 356]
```

**注意：** `path` 包里包含一个子包叫 `filepath`，这个子包提供了跨平台的函数，用于处理文件名和路径。例如 Base() 函数用于获得路径中的最后一个元素（不包含后面的分隔符）：  

```go
import "path/filepath"
filename := filepath.Base(path)
```

### `compress` 包：读取压缩文件  
`compress`包提供了读取压缩文件的功能，支持的压缩文件格式为：bzip2、flate、gzip、lzw 和 zlib。  

下面的程序展示了如何读取一个 gzip 文件。

```go
package main

import (
    "fmt"
    "bufio"
    "os"
    "compress/gzip"
)

func main() {
    fName := "MyFile.gz"
    var r *bufio.Reader
    fi, err := os.Open(fName)
    if err != nil {
        fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", os.Args[0], fName,
            err)
        os.Exit(1)
    }
    defer fi.Close()
    fz, err := gzip.NewReader(fi)
    if err != nil {
        r = bufio.NewReader(fi)
    } else {
        r = bufio.NewReader(fz)
    }

    for {
        line, err := r.ReadString('\n')
        if err != nil {
            fmt.Println("Done reading file")
            os.Exit(0)
        }
        fmt.Println(line)
    }
}
```

### 写文件  
```go
package main

import (
    "os"
    "bufio"
    "fmt"
)

func main () {
    // var outputWriter *bufio.Writer
    // var outputFile *os.File
    // var outputError os.Error
    // var outputString string
    outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
    if outputError != nil {
        fmt.Printf("An error occurred with file opening or creation\n")
        return  
    }
    defer outputFile.Close()

    outputWriter := bufio.NewWriter(outputFile)
    outputString := "hello world!\n"

    for i:=0; i<10; i++ {
        outputWriter.WriteString(outputString)
    }
    outputWriter.Flush()
}
```

除了文件句柄，我们还需要 `bufio` 的 `Writer`。我们以只写模式打开文件 `output.dat`，如果文件不存在则自动创建：  

```go
outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
```

可以看到，`OpenFile` 函数有三个参数：文件名、一个或多个标志（使用逻辑运算符“|”连接），使用的文件权限。  

我们通常会用到以下标志：  

- `os.O_RDONLY`：只读  
- `os.O_WRONLY`：只写  
- `os.O_CREATE`：创建：如果指定文件不存在，就创建该文件。  
- `os.O_TRUNC`：截断：如果指定文件已存在，就将该文件的长度截为0。

在读文件的时候，文件的权限是被忽略的，所以在使用 `OpenFile` 时传入的第三个参数可以用0。而在写文件时，不管是 Unix 还是 Windows，都需要使用 0666。  

然后，我们创建一个写入器（缓冲区）对象：  

```go
outputWriter := bufio.NewWriter(outputFile)
```

接着，使用一个 for 循环，将字符串写入缓冲区，写 10 次：`outputWriter.WriteString(outputString)`  

缓冲区的内容紧接着被完全写入文件：`outputWriter.Flush()`  

如果写入的东西很简单，我们可以使用 `fmt.Fprintf(outputFile, "Some test data.\n")` 直接将内容写入文件。`fmt` 包里的 F 开头的 Print 函数可以直接写入任何 `io.Writer`，包括文件。  

程序 `filewrite.go` 展示了不使用 `fmt.FPrintf` 函数，使用其他函数如何写文件：  


```go
package main

import "os"

func main() {
    os.Stdout.WriteString("hello, world\n")
    f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)
    defer f.Close()
    f.WriteString("hello, world in a file\n")
}
```

使用 `os.Stdout.WriteString("hello, world\n")`，我们可以输出到屏幕。  

我们以只写模式创建或打开文件"test"，并且忽略了可能发生的错误：`f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)`  

我们不使用缓冲区，直接将内容写入文件：`f.WriteString( )`  


---
# 常用资料  
## Go 关键字
* 25 个关键字  
```go
break      default       func     interface   select
case       defer         go       map         struct
chan       else          goto     package     switch
const      fallthrough   if       range       type
continue   for           import   return      var
```

* 36 个预定义标识符  

```go
内建常量: true false iota nil

内建类型: int int8 int16 int32 int64
          uint uint8 uint16 uint32 uint64 uintptr
          float32 float64 complex128 complex64
          bool byte rune string error

内建函数: make len cap new append copy close delete
          complex real imag
          panic recover
```






---
*`to be continued...`*
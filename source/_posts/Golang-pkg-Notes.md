---
title: Golang pkg Notes
date: 2019-03-20 22:16:19
tags: Go
categories: Go
permalink: Golang-pkg
---
# `bytes` 包
对于传入 []byte 的函数，都不会修改传入的参数，返回值要么是参数的副本，要么是参数的切片。  

## 转换  
### 将s中所有字符修改为大写（小写、标题）格式返回  
```go
func ToUpper(s []byte) []byte
func ToLower(s []byte) []byte
func ToTitle(s []byte) []byte
```
### 使用指定的映射表将 s 中的所有字符修改为大写（小写、标题）格式返回  
```go
func ToUpperSpecial(_case unicode.SpecialCase, s []byte) []byte
func ToLowerSpecial(_case unicode.SpecialCase, s []byte) []byte
func ToTitleSpecial(_case unicode.SpecialCase, s []byte) []byte
```
### 将 s 中的所有单词的首字符修改为 Title 格式返回  
```go
func Title(s []byte) []byte
```
BUG: 不能很好的处理以 Unicode 标点符号分隔的单词  

## 比较  
### 比较两个[]byte的字典顺序  
nil 参数相当于空[]byte
```go
func Compare(a, b []byte) int
```
a <  b 返回 -1
a == b 返回 0
a >  b 返回 1

### 判断a、b是否相等  
nil 相当于空[]byte  
```go
func Equal(a, b []byte) bool
```

### 判断s，t是否相似  
忽略大写、小写、标题三种格式的区别。  
参考 `unicode.SimpleFold` 函数  
```go
func EqualFold(s, t []byte) bool
```

## 清理  
### 去掉 s 左右两边包含在cutset中的字符（返回s的切片）  
```go
func Trim(s []byte, cutset string) []byte
func TrimLeft(s []byte, cutset string) []byte
func TrimRight(s []byte, cutset string) []byte
```
### 去掉s两边符合f要求的字符（返回s的切片）  
```go
func TrimFunc(s []byte, f func(r rune) bool) []byte
func TrimLeftFunc(s []byte, f func(r rune) bool) []byte
func TrimRightFunc(s []byte, f func(r rune) bool) []byte
```
### 去掉s两边的空白(unicode.IsSpace)（返回s的切片）  
```go
func TrimSpace(s []byte) []byte
```
### 去掉 s 的前缀 prefix（后缀 suffix）（返回 s 的切片）  
```go
func TrimPrefix(s, prefix []byte) []byte
func TrimSuffix(s, suffix []byte) []byte
```

## 拆合  
### Split按seq分割  
Split 以 sep 为分隔符将 s 切分成多个子串，结果不包含分隔符。  
如果 sep 为空，则将 s 切分成 Unicode 字符列表。  
SplitN 可以指定切分次数 n，超出 n 的部分将不进行切分。  
```go
func Split(s, sep []byte) [][]byte
func SplitN(s, sep []byte, n int) [][]byte
```
### SplitAfter  
功能同 Split，只不过结果包含分隔符（在各个子串尾部）。  
```go
func SplitAfter(s, sep []byte) [][]byte
func SplitAfterN(s, sep []byte, n int) [][]byte
```
### Fields 空白符分割  
以连续空白为分隔符将 s 切分成多个子串，结果不包含分隔符。  
```go
func Fields(s []byte) [][]byte
```
### 以符合 f 的字符为分隔符将 s 切分成多个子串，结果不包含分隔符  
```go
func FieldsFunc(s []byte, f func(rune) bool) [][]byte
```
### Join，sep为连接符  
```go
func Join(s [][]byte, sep []byte) []byte
```
### 把子串b重复count次吼返回  
```go
func Repeat(b []byte, count int) []byte
```
eg.
```go
func main() {
    b := []byte("  Hello   World !  ")
    fmt.Printf("%q\n", bytes.Split(b, []byte{' '}))
    // ["" "" "Hello" "" "" "World" "!" "" ""]
    fmt.Printf("%q\n", bytes.Fields(b))
    // ["Hello" "World" "!"]
    f := func(r rune) bool {
        return bytes.ContainsRune([]byte(" !"), r)
    }
    fmt.Printf("%q\n", bytes.FieldsFunc(b, f))
    // ["Hello" "World"]
}
```

## 子串  
### 判断前缀、后缀  
```go
func HasPrefix(s, prefix []byte) bool
func HasSuffix(s, suffix []byte) bool
```
### 是否包含子串subslice 或字符r  
```go
func Contains(b, subslice []byte) bool
func ContainsRune(b []byte, r rune) bool
```
### 是否包含chars中任一字符
```go
func ContainsAny(b []byte, chars string) bool
```
### 查找子串seq（字节c、字符r）在s中第一次出现的位置  
找不到则返回-1  
```go
func Index(s, sep []byte) int
func IndexByte(s []byte, c byte) int
func IndexRune(s []byte, r rune) int
```
### 查找chars中任一字符在s中第一次出现的位置  
找不到则返回-1  
```go
func IndexAny(s []byte, chars string) int
```
### 查找符合f的字符在s中第一次出现的位置  
找不到则返回-1  
```go
func IndexFunc(s []byte, f func(r rune) bool) int
```
### 功能同上，只不过查找最后一次出现的位置  
```go
func LastIndex(s, sep []byte) int
func LastIndexByte(s []byte, c byte) int
func LastIndexAny(s []byte, chars string) int
func LastIndexFunc(s []byte, f func(r rune) bool) int
```
### 获取seq在s中出现的次数（不重叠）  
```go
func Count(s, sep []byte) int
```

## 替换  
### s中前n个old替换为new，n<0则替换全部  
```go
func Replace(s, old, new []byte, n int) []byte
```
### 将 s 中的字符替换为 mapping(r) 的返回值  
如果 mapping 返回负值，则丢弃该字符  
```go
func Map(mapping func(r rune) rune, s []byte) []byte
```
### 将s转换为[]rune类型返回
```go
func Runes(s []byte) []rune
```

## type  
### type Reader  
```go
type Reader struct { ... }
```
将 b 包装成 bytes.Reader 对象。  
```go
func NewReader(b []byte) *Reader
```
bytes.Reader 实现了如下接口：  
```
io.ReadSeeker
io.ReaderAt
io.WriterTo
io.ByteScanner
io.RuneScanner
```
返回未读取部分的数据长度：  
```go
func (r *Reader) Len() int
```
返回底层数据的总长度，方便ReadAt使用，返回值永远不变：  
```go
func (r *Reader) Size() int64
```
将底层数据切换为 b，同时复位所有标记（读取位置等信息）  
```go
func (r *Reader) Reset(b []byte)
```
eg.
```go
func main() {
    b1 := []byte("Hello World!")
    b2 := []byte("Hello 世界！")
    buf := make([]byte, 6)
    rd := bytes.NewReader(b1)
    rd.Read(buf)
    fmt.Printf("%q\n", buf) // "Hello "
    rd.Read(buf)
    fmt.Printf("%q\n", buf) // "World!"

    rd.Reset(b2)
    rd.Read(buf)
    fmt.Printf("%q\n", buf) // "Hello "
    fmt.Printf("Size:%d, Len:%d\n", rd.Size(), rd.Len())
    // Size:15, Len:9
}
```

### type Buffer  
```go
type Buffer struct { ... }
```
将 buf 包装成 bytes.Buffer 对象：  
```go
func NewBuffer(buf []byte) *Buffer
```
将 s 转换为 []byte 后，包装成 bytes.Buffer 对象：  
```go
func NewBufferString(s string) *Buffer
```
Buffer 本身就是一个缓存（内存块），没有底层数据，缓存的容量会根据需要自动调整。大多数情况下，使用 new(Buffer) 就足以初始化一个 Buffer 了。  

bytes.Buffer 实现了如下接口：  
```
io.ReadWriter
io.ReaderFrom
io.WriterTo
io.ByteWeriter
io.ByteScanner
io.RuneScanner
```
未读取部分的数据长度：  
```go
func (b *Buffer) Len() int
```
缓存的容量：  
```go
func (b *Buffer) Cap() int
```
读取前 n 字节的数据并以切片形式返回，如果数据长度小于 n，则全部读取。  
切片只在下一次读写操作前合法。  
```go
func (b *Buffer) Next(n int) []byte
```
读取第一个 delim 及其之前的内容，返回遇到的错误（一般是 io.EOF）：  
```go
func (b *Buffer) ReadBytes(delim byte) (line []byte, err error)
func (b *Buffer) ReadString(delim byte) (line string, err error)
```
写入 r 的 UTF-8 编码，返回写入的字节数和 nil。  
保留 err 是为了匹配 bufio.Writer 的 WriteRune 方法。  
```go
func (b *Buffer) WriteRune(r rune) (n int, err error)
```
写入 s，返回写入的字节数和 nil：  
```go
func (b *Buffer) WriteString(s string) (n int, err error)
```
引用未读取部分的数据切片（不移动读取位置）：  
```go
func (b *Buffer) Bytes() []byte
```
返回未读取部分的数据字符串（不移动读取位置）：  
```go
func (b *Buffer) String() string
```
自动增加缓存容量，以保证有 n 字节的剩余空间。  
如果 n 小于 0 或无法增加容量则会 panic。  
```go
func (b *Buffer) Grow(n int)
```
将数据长度截短到 n 字节，如果 n 小于 0 或大于 Cap 则 panic：  
```go
func (b *Buffer) Truncate(n int)
```
重设缓冲区，清空所有数据（包括初始内容）：  
```go
func (b *Buffer) Reset()
```
eg.  
```go
func main() {
    rd := bytes.NewBufferString("Hello World!")
    buf := make([]byte, 6)
    // 获取数据切片
    b := rd.Bytes()
    // 读出一部分数据，看看切片有没有变化
    rd.Read(buf)
    fmt.Printf("%s\n", rd.String()) // World!
    fmt.Printf("%s\n\n", b)         // Hello World!

    // 写入一部分数据，看看切片有没有变化
    rd.Write([]byte("abcdefg"))
    fmt.Printf("%s\n", rd.String()) // World!abcdefg
    fmt.Printf("%s\n\n", b)         // Hello World!

    // 再读出一部分数据，看看切片有没有变化
    rd.Read(buf)
    fmt.Printf("%s\n", rd.String()) // abcdefg
    fmt.Printf("%s\n", b)           // Hello World!
}
```

---
# `fmt` 包  
实现了格式化I/O函数。[文档](http://docscn.studygolang.com/pkg/fmt/)  
## 占位符  
通用占位符：  
```
%v  值的默认格式。当打印结构体时，“加号”标记（%+v）会添加字段名
%#v　相应值的Go语法表示
%T  相应值的类型的Go语法表示
%%  字面上的百分号，并非值的占位符　
```
```go

package main
 
import (
    "fmt"
)
 
type Sample struct {
    a   int
    str string
}
 
func main() {
    s := new(Sample)
    s.a = 1
    s.str = "hello"
    fmt.Printf("%v\n", *s)　//{1 hello}
    fmt.Printf("%+v\n", *s) //  {a:1 str:hello}
    fmt.Printf("%#v\n", *s) // main.Sample{a:1, str:"hello"}
    fmt.Printf("%T\n", *s)   //  main.Sample
    fmt.Printf("%%\n", s.a) //  %  %!(EXTRA int=1)
}
```
布尔值：  
```
%t   true 或 false
```
整数值：  
```
%b  二进制表示
%c  相应Unicode码点所表示的字符
%d  十进制表示
%o  八进制表示
%q  单引号围绕的字符字面值，由Go语法安全地转义
%x  十六进制表示，字母形式为小写 a-f
%X  十六进制表示，字母形式为大写 A-F
%U  Unicode格式：U+1234，等同于 "U+%04X"
```
浮点及复数：  
```
%b  无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat中的 'b' 转换格式一致。例如 -123456p-78
%e  科学计数法，例如 -1234.456e+78
%E  科学计数法，例如 -1234.456E+78
%f  有小数点而无指数，例如 123.456
%g  根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出
%G  根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的0）输出
```
字符串和bytes的slice表示：
```
%s  字符串或切片的无解译字节
%q  双引号围绕的字符串，由Go语法安全地转义
%x  十六进制，小写字母，每字节两个字符
%X  十六进制，大写字母，每字节两个字符
```
指针：  
```
%p  十六进制表示，前缀 0x
```
这里没有 'u' 标记。若整数为无符号类型，他们就会被打印成无符号的。类似地，这里也不需要指定操作数的大小（int8，int64）。  

对于％ｖ来说默认的格式是：  
```
bool:                    %t
int, int8 etc.:          %d
uint, uint8 etc.:        %d, %x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p
pointer:                 %p
```
对于复合对象，里面的元素使用如下规则进行打印：  
```
struct:             {field0 field1 ...}
array, slice:       [elem0  elem1 ...]
maps:               map[key1:value1 key2:value2]
pointer to above:   &{}, &[], &map[]
```
宽度和精度：  
宽度是在％之后的值，如果没有指定，则使用该值的默认值，精度是跟在宽度之后的值，如果没有指定，也是使用要打印的值的默认精度．例如：％９.２f，宽度９，精度２  
```
%f:    default width, default precision
%9f    width 9, default precision
%.2f   default width, precision 2
%9.2f  width 9, precision 2
%9.f   width 9, precision 0
```
对数值而言，宽度为该数值占用区域的最小宽度；精度为小数点之后的位数。但对于 %g/%G 而言，精度为所有数字的总数。例如，对于123.45，格式 %6.2f会打印123.45，而 %.4g 会打印123.5。%e 和 %f 的默认精度为6；但对于 %g 而言，它的默认精度为确定该值所必须的最小位数。  

对大多数值而言，宽度为输出的最小字符数，如果必要的话会为已格式化的形式填充空格。对字符串而言，精度为输出的最大字符数，如果必要的话会直接截断。  

宽度是指"必要的最小宽度". 若结果字符串的宽度超过指定宽度时, 指定宽度就会失效。  

其他标志：  
```
+   总打印数值的正负号；对于%q（%+q）保证只输出ASCII编码的字符。
-   左对齐
#   备用格式：为八进制添加前导 0（%#o），为十六进制添加前导 0x（%#x）或0X（%#X），为 %p（%#p）去掉前导 0x；对于 %q，若 strconv.CanBackquote
    返回 true，就会打印原始（即反引号围绕的）字符串；如果是可打印字符，%U（%#U）会写出该字符的Unicode编码形式（如字符 x 会被打印成 U+0078 'x'）。
' ' （空格）为数值中省略的正负号留出空白（% d）；以十六进制（% x, % X）打印字符串或切片时，在字节之间用空格隔开
0   填充前导的0而非空格；对于数字，这会将填充移到正负号之后
```
对于每一个 Printf 类的函数，都有一个 Print 函数，该函数不接受任何格式化，它等价于对每一个操作数都应用 %v。另一个变参函数 Println 会在操作数之间插入空白，并在末尾追加一个换行符  

格式化错误：  

如果给占位符提供了无效的实参（如将一个字符串提供给％d），便会出现格式化错误．所有的错误都始于“%!”，有时紧跟着单个字符（占位符），并以小括号括住的描述结尾。  

## Scanning  
一组类似的函数通过扫描已格式化的文本来产生值。  

Scan、Scanf 和 Scanln 从os.Stdin 中读取；  
Fscan、Fscanf 和 Fscanln 从指定的 io.Reader 中读取；  
Sscan、Sscanf 和 Sscanln 从实参字符串中读取。  
Scanln、Fscanln 和 Sscanln在换行符处停止扫描，且需要条目紧随换行符之后；  
Scanf、Fscanf 和 Sscanf需要输入换行符来匹配格式中的换行符；  
其它函数则将换行符视为空格。  

Scanf、Fscanf 和 Sscanf 根据格式字符串解析实参，类似于 Printf。例如，%x会将一个整数扫描为十六进制数，而 %v 则会扫描该值的默认表现格式。  

格式化类似于 Printf，但也有例外，如下所示：  
```
%p 没有实现
%T 没有实现
%e %E %f %F %g %G 都完全等价，且可扫描任何浮点数或复合数值
%s 和 %v 在扫描字符串时会将其中的空格作为分隔符
标记 # 和 + 没有实现
```
在输入Scanf中，宽度可以理解成输入的文本（％5s表示输入５个字符），而Scanf没有精度这种说法（没有%5.2f，只有 %5f）  

## 函数  
### Errorf  
```go
func Errorf(format string, a ...interface{}) error
```
Errorf 根据于格式说明符进行格式化，并将字符串作为满足 error 的值返回，其返回类型是error(功能同Sprintf，只不过把结果字符串包装成error类型).  
```go
func main() {
    a := fmt.Errorf("%s%d", "error:", 1)
    fmt.Println(a)
```
### Fprint、Fprintf、Fprintln  
对于每一个 Printf 类的函数，都有一个 Print 函数，该函数不接受任何格式化，它等价于对每一个操作数都应用 %v。另一个变参函数 Println 会在操作数之间插入空白，并在末尾追加一个换行符  
```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)　
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) 
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```
Fprint 使用其操作数的默认格式进行格式化并写入到 w。当两个连续的操作数均不为字符串时，它们之间就会添加空格。它返回写入的字节数以及任何遇到的错误。  
Fprintf 根据于格式说明符进行格式化并写入到 w。它返回写入的字节数以及任何遇到的写入错误。  
Fprintln 使用其操作数的默认格式进行格式化并写入到 w。其操作数之间总是添加空格，且总在最后追加一个换行符。它返回写入的字节数以及任何遇到的错误。  
```go
func main() {
    a := "asdf"
    fmt.Fprintln(os.Stdout, a)          //asdf
    fmt.Fprintf(os.Stdout, "%.2s\n", a) //as
    fmt.Fprint(os.Stdout, a)            //asdf
}
```
### Fscan、Fscanf、Fscanln  
```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error) 
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
```
Fscan 扫描从 r 中读取的文本，并将连续由空格分隔的值存储为连续的实参。换行符计为空格。它返回成功扫描的条目数。若它少于实参数，err 就会报告原因。  
Fscanf 扫描从 r 中读取的文本，并将连续由空格分隔的值存储为连续的实参，其格式由 format 决定。它返回成功解析的条目数。  
Fscanln 类似于 Sscan，但它在换行符处停止扫描，且最后的条目之后必须为换行符或 EOF。  
注：Fscan类的也是由空格进行分割的．  
```go

func main() {
    r := strings.NewReader("hello 1")
    var a string
    var b int
    fmt.Fscanln(r, &a, &b)
    fmt.Println(a, b)　　　　　　　　 //hello 1
    r1 := strings.NewReader("helloworld 2")
    fmt.Fscanf(r1, "hello%s%d", &a, &b) 　
    fmt.Println(a, b)　　　　　　　　//world 2
}
```
### Print、Printf、Println  
```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error) 
func Println(a ...interface{}) (n int, err error)
```
Print 使用其操作数的默认格式进行格式化并写入到标准输出。当两个连续的操作数均不为字符串时，它们之间就会添加空格。它返回写入的字节数以及任何遇到的错误。  
Printf 根据格式说明符进行格式化并写入到标准输出。它返回写入的字节数以及任何遇到的写入错误。  
使用其操作数的默认格式进行格式化并写入到标准输出。其操作数之间总是添加空格，且总在最后追加一个换行符。它返回写入的字节数以及任何遇到的错误。  
```go

func main() {
    s := "hello,world!"
    fmt.Println(s)     //hello,world!
    fmt.Printf("%s\n", s)    //hello,world!
    fmt.Print(s)           //hello,world!
}
```
类似于 `Fprint(os.Stdout,...)`  

### Scan、Scanf、Scanln  
```go
func Scan(a ...interface{}) (n int, err error)
func Scanf(format string, a ...interface{}) (n int, err error)
func Scanln(a ...interface{}) (n int, err error)
```
Scan 扫描从标准输入中读取的文本，并将连续由空格分隔的值存储为连续的实参。换行符计为空格。它返回成功扫描的条目数。若它少于实参数，err 就会报告原因。  
Scanf 扫描从标准输入中读取的文本，并将连续由空格分隔的值存储为连续的实参，其格式由 format 决定。它返回成功扫描的条目数。  
Scanln 类似于 Scan，但它在换行符处停止扫描，且最后的条目之后必须为换行符或 EOF。

```go
func main() {
    var a string
    var b int
    fmt.Scanln(&a, &b)   // 2,1
    fmt.Println(a, b)        //输出２　１
    fmt.Scanf("%s%d", &a, &b)　//2 1
    fmt.Println(a, b)　　//输出２　１
}
```
### Sprint、Sprintf、Sprintln  
```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```
Sprint 使用其操作数的默认格式进行格式化并返回其结果字符串。当两个连续的操作数均不为字符串时，它们之间就会添加空格。  
Sprintf 根据于格式说明符进行格式化并返回其结果字符串。  
Sprintln 使用其操作数的默认格式进行格式化并写返回其结果字符串。其操作数之间总是添加空格，且总在最后追加一个换行符。  
```go
func main() {
    a := fmt.Sprintf("%s,%d", "hello", 1)
    fmt.Println(a)       //hello,1
}
```

### Sscan、Sscanf、Sscanln  
```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error) 
```
Sscan 扫描实参 string，并将连续由空格分隔的值存储为连续的实参。换行符计为空格。它返回成功扫描的条目数。若它少于实参数，err 就会报告原因。  
Scanf 扫描实参 string，并将连续由空格分隔的值存储为连续的实参，其格式由 format 决定。它返回成功解析的条目数。  
Sscanln 类似于 Sscan，但它在换行符处停止扫描，且最后的条目之后必须为换行符或 EOF。
```go
func main() {
    var a string
    var b int
    var c int
    fmt.Sscan("hello 1", &a, &b) //hello 1
    fmt.Println(a, b)
    fmt.Sscanf("helloworld 2 ", "hello%s%d", &a, &c) //world 2
    fmt.Println(a, c)
}
```
## type  
### type Formatter  
用于实现对象的自定义格式输出  
```go
type Formatter interface {
    // Format 用来处理当对象遇到 c 标记时的输出方式（c 相当于 %s 中的 s）
    // f 用来获取占位符的宽度、精度、扩展标记等信息，同时实现最终的输出
    // c 是要处理的标记
    Format(f State, c rune)
}
```
### type GoStringer  
```go
type GoStringer interface {
    // GoString 获取对象的 Go 语法文本形式（以 %#v 格式输出的文本）
    GoString() string
}
```
### type ScanState  
ScanState 会返回扫描状态给自定义的 Scanner  
Scanner 可能会做字符的实时扫描  
或者通过 ScanState 获取以空格分割的 token  
```go
type ScanState interface {
    // ReadRune 从输入对象中读出一个 Unicode 字符
    //world 2
}
```
如果在 Scanln、Fscanln 或 Sscanln 中调用该方法  
该方法会在遇到 '\n' 或读取超过指定的宽度时返回 EOFReadRune() (r rune, size int, err error)  
UnreadRune 撤消最后一次的 ReadRune 操作UnreadRune() error  
SkipSpace 跳过输入数据中的空格  
在 Scanln、Fscanln、Sscanln 操作中，换行符会被当作 EOF  
在其它 Scan 操作中，换行符会被当作空格SkipSpace()  
如果参数 skipSpace 为 true，则 Token 会跳过输入数据中的空格  
然后返回满足函数 f 的连续字符，如果 f 为 nil，则使用 !unicode.IsSpace 来代替 f  
在 Scanln、Fscanln、Sscanln 操作中，换行符会被当作 EOF  
在其它 Scan 操作中，换行符会被当作空格  
返回的 token 是一个切片，返回的数据可能在下一次调用 Token 的时候被修改Token(skipSpace bool, f func(rune) bool) (token []byte, err error)  
Width 返回宽度值以及宽度值是否被设置Width() (wid int, ok bool)  
因为 ReadRune 已经通过接口实现，所以 Read 可能永远不会被 Scan 例程调用  
一个 ScanState 的实现，可能会选择废弃 Read 方法，而使其始终返回一个错误信息Read(buf []byte) (n int, err error)}

### type Scanner  
Scanner 由任何拥有 Scan 方法的值实现，它将输入扫描成值的表示，并将其结果存储到接收者中，该接收者必须为可用的指针。Scan 方法会被 Scan、Scanf 或 Scanln 的任何实现了它的实参所调用。  
```go
type Scanner interface {
    Scan(state ScanState, verb rune) error
}
```

### type State  
```go
type State interface {

    // Write 函数用于打印出已格式化的输出。
    Write(b []byte) (ret int, err error)

    // Width 返回宽度选项的值以及它是否已被设置。
    Width() (wid int, ok bool)

    // Precision 返回精度选项的值以及它是否已被设置。
    Precision() (prec int, ok bool)

    // Flag 返回标记 c（一个字符）是否已被设置。
    Flag(c int) bool
}
```

### type Stringer  
```go
type Stringer interface {
    // String 获取对象的文本形式
    String() string
}
```

---

# `os` 包
## 函数  
### 获取当前目录、改变目录  
```go
func Getwd() (dir string, err error)    //获取当前目录
func Chdir(dir string) error            // 修改当前目录

eg.
func main() {
  fmt.Println(os.Getwd())
  os.Chdir("/Users/rtk")
  fmt.Println(os.Getwd())
}
```
### 修改文件权限  
```go
func Chmod(name string, mode FileMode) error

eg. 将文件改为不可读：
func main() {
  fmt.Println(os.Getwd())
  os.Chdir("/Users/xujie/Desktop")
  os.Chmod("file.txt",0)
}
```
### 获取用户识别码uid 和 群组识别码gid  
```go
func Getuid()  int 
func Getegid()  int  // 有效的用户识别码
func Geteuid() int  // 有效的群组识别码
func Getgid()  int 
```

### 查看用户所属组的列表  
```go
func Getgroups() ([]int, error)
```

### 返回底层系统的内存页面大小  
```go
func Getpagesize() int
```

### 获取主机名称  
```go
func Hostname() (name string, err error)
```

### 获取当前进程id、父进程id  
```go
func Getpid()       // 获取当前进程id
func Getppid()      // 获取父进程id
```

### 获取文件的状态  
```go
func Stat(name string) (FileInfo, error)

eg.
func main() {
  filename := "/Users/xujie/Desktop/file.txt"
  file,_:=os.Stat(filename)
  fmt.Println(file.Name())
  fmt.Println(file.IsDir())
  fmt.Println(file.Size())
  fmt.Println(file.Mode())
  fmt.Println(file.Sys())
}
```

### 错误检测  
```go
func IsExist(err error) bool        // 文件存在,但是由系统产生错误
func IsNotExist(err error) bool     // 目录或者文件不存在时返回true
func IsPermission(err error)bool    // 检测是不是由于权限不足导致的错误

eg.
func main() {
  filename := "file.txt"
  _, err := os.Stat(filename);
  fmt.Println(os.IsExist(err))
  fmt.Println(os.IsNotExist(err))
}
```

### 创建文件夹、删除文件或文件夹  
```go
func Mkdir(name string, perm FileMode) error

func Remove(name string) error
func RemoveAll(name string) error   // 删除文件夹下所有文件
```

### 修改文件夹或文件的名称  
```go
func Rename(oldpath, newpath string) error
```

### 移动文件夹或文件  
```go
func Rename(oldpath, newpath string) error
```

### 新建文件  
```go
func Create(name string) (*File, error)
```
这个方法创建文件是默认的权限为0666,如果已经存在同名的文件,则调用此方法,会覆盖掉原来的文件  

### 打开文件  
```go
func Open(name string) (*File, error)
```
如果打开的文件不存在,则会返回错误 `open file1.txt: no such file or directory`  

### 写入文件  
```go
func (f *File) Write(b []byte) (n int, err error)
func (f *File) WriteAt(b []byte, off int64) (n int, err error)
func (f *File) WriteString(s string) (n int, err error)
```
要注意的是，os.open() 打开的文件是只读的，写入不成功，需要：  
```go
func OpenFile(name string, flag int, perm FileMode) (*File, error)

第二个参数
O_RDONLY    打开只读文件
O_WRONLY    打开只写文件
O_RDWR  打开既可以读取又可以写入文件
O_APPEND    写入文件时将数据追加到文件尾部
O_CREATE    如果文件不存在，则创建一个新的文件
O_EXCL  文件必须不存在，然后会创建一个新的文件
O_SYNC  打开同步I/0
O_TRUNC 文件打开时可以截断

第三个参数就是权限模式
```
例子：  
```go
func main() {
  // 进入桌面目录
  os.Chdir("/Users/xujie/Desktop")
  // 创建一个文件夹
 file,error:= os.OpenFile("file.txt",os.O_RDWR,0666)
 // 如果是要追加内容：
 // file,error:= os.OpenFile("file.txt",os.O_RDWR|os.O_APPEND,0666)
 defer file.Close()
 if error != nil {
   fmt.Println(error)
 }
 _,error = file.WriteString("你好")
  _,error = file.WriteString("从天有一个书")
  fmt.Println(error)
}
```

### 读取文件  
```go
func main() {
  // 进入桌面目录
  os.Chdir("/Users/xujie/Desktop")
 file,error:= os.Open("file.txt")

 defer file.Close()
 if error != nil {
   fmt.Println(error)
   return
 }
  fileInfo,_ := file.Stat()
  fmt.Println(fileInfo.Size())
  
  // 创建缓冲区
 data := make([]byte,fileInfo.Size())
 // 一次性读取所有内容到缓冲区中
 _,error = file.Read(data)

 fmt.Println(error)
 fmt.Println(string(data))
}
```

### 关闭文件  
```go
func (f *File) Close() error

defer file.Close()
```

### 检测文件是否是同一个  
```go
func SameFile(fi1, fi2 FileInfo) bool
```
就算是同一个文件,没在同一个路径下也会返回false，判断依据如下  
这意味着两个基础结构的 inode 字段是相同的  

### 获取文件模式相关信息  
```go
func (m FileMode) IsDir() bool
func (m FileMode) IsRegular() bool
func (m FileMode) Perm() FileMode
func (m FileMode) String() string

eg.
func main() {
  fileInfo1,_:= os.Stat("/Users/xujie/Desktop/DataParser.zip")
  fmt.Println(fileInfo1.Mode().String())
  fmt.Println(fileInfo1.Mode().IsRegular())
  fmt.Println(fileInfo1.Mode().IsDir())
  fmt.Println(fileInfo1.Mode().Perm())
}
```

### 把文件所在的目录切换为当前目录  
```go
func (f *File) Chdir() error
```

### 查看文件名称  
```go
func (f *File) Name() string
```

### 如何查看所有文件夹下的所有文件和文件数量  
```go
func (f *File) Readdir(n int) ([]FileInfo, error)
// n 表示读取目录下文件的最大数量,n < 0 表示全部 

eg.
func main() {
  file,_ := os.Open("/Users/xujie/Desktop/未命名文件夹")
  files,_ := file.Readdir(10)
  for _,f := range  files {
    fmt.Println(f.Name())
  }
}
```

### 读取文件夹的下面文件的名称  
```go
func (f *File) Readdirnames(n int) (names []string, err error)

eg:
func main() {
  file,_ := os.Open("/Users/xujie/Desktop/未命名文件夹")
  fileNames,_ := file.Readdirnames(-1)
  for _,name := range  fileNames {
    fmt.Println(name)
  }
}
```

### 获取临时陌路的文件夹路径  
```go
func TempDir() string
```

### 判断字符是否是支持的路径分隔符  
```go
func IsPathSeparator(c uint8) bool

eg：
func main() {
 fmt.Println(os.IsPathSeparator('c'))
  fmt.Println(os.IsPathSeparator('\\'))
  fmt.Println(os.IsPathSeparator('/'))
}
```

### 查看环境变量  
```go
func Getenv(key string) string

func LookupEnv(key string) (string, bool)
//key区分大小写
```

### 查找指定环境变量  
```go
func func Expand(s string, mapping func(string) string) string

func ExpandEnv(s string) string
```

### 获取文件对应的unix文件描述符  
```go
func (f *File) Fd() uintptr
```

### Chown修改文件的用户ID和组ID  
```go
func Chown(name string, uid, gid int) error
```

### 修改文件权限  
```go
func (f *File) Chmod(mode FileMode) error
```

### 强制改变文件大小  
两个方法：  
```go
func Truncate(name string, size int64) error

func (f *File)Truncate(size int64) error
```
减小文件大小会造成内容丢失。  

### 链接 硬链接  
硬链接(hard link, 也称链接)就是一个文件的一个或多个文件名。再说白点，所谓链接无非是把文件名和计算机文件系统使用的节点号链接起来。因此我们可以用多个文件名与同一个文件进行链接，这些文件名可以在同一目录或不同目录  
```go
func Link(oldname, newname string) error
```

### 同步保存当前文件的内容  
```go
func (f *File) Sync() error
```
Sync递交文件的当前内容进行稳定的存储。一般来说，这表示将文件系统的最近写入的数据在内存中的拷贝刷新到硬盘中稳定保存  
```go
func main() {
  file,error := os.OpenFile("/Users/xujie/Desktop/file.txt",os.O_RDWR,0777)
  if error != nil{
   fmt.Println(error)
  }
  file.WriteString("新内容")
  file.Sync()
}
```

### NewFile使用给出的Unix文件描述符和名称创建一个文件  
```go
func NewFile(fd uintptr, name string) *File

```

### Lstat返回一个描述name指定的文件对象的FileInfo  
```go
func Lstat(name string) (fi FileInfo, err error)
```
Lstat返回一个描述name指定的文件对象的FileInfo。如果指定的文件对象是一个符号链接，返回的FileInfo描述该符号链接的信息，本函数不会试图跳转该链接。如果出错，返回的错误值为*PathError类型   

### 查看所有环境变量、清除环境变量  
```go
func Environ() []string
func Clearenv() []string  //慎用！！
```

### 获取当前程序可执行的文件地址  
```go
func Executable() (string, error)
```

### 让程序已给定的状态码退出  
Exit让当前程序以给出的状态码code退出。一般来说，状态码0表示成功，非0表示出错。程序会立刻终止，defer的函数不会被执行  
```go
func Exit(code int)
```

### 设置环和取消环境变量  
```go
func Unsetenv(key string) error

func Setenv(key, value string) error

eg：
func main() {
    os.Setenv("TMPDIR", "/my/tmp")
    defer os.Unsetenv("TMPDIR")
}
```

### 创建软链接  
[软连接和硬链接的区别](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/index.html)  
```go
func Symlink(oldname, newname string) error
```

### 获取软链接文件对应的实际文件路径地址  
```go
func Readlink(name string) (string, error)
```

### 更改指定文件的访问和修改时间  
```go
func Chtimes(name string, atime time.Time, mtime time.Time) error
```
第二个参数访问时间 第三个参数修改时间  
```go
 os.Chtimes("/Users/xujie/Desktop/file.txt",time.Now(),time.Now().Add(time.Hour * 24))
```

### 创建文件夹,并设置权限  
```go
func MkdirAll(path string, perm FileMode) error
```
规则：  
- 如果已经存在同名文件夹,则此方法不做任何事情
- 文件夹里面所有文件都是同样的权限

### 设置文章的读写位置  
```go
func (f *File) Seek(offset int64, whence int) (ret int64, err error)
```
Seek设置下一次读/写的位置。offset为相对偏移量，而whence决定相对位置：0为相对文件开头，1为相对当前位置，2为相对文件结尾。它返回新的偏移量（相对开头）和可能的错误  

```go
eg：
func main() {

 file,error := os.Open("/Users/xujie/Desktop/file.txt")
 defer  file.Close()
 if error != nil {
   fmt.Println(error)
   return
 }

 offset := int64(0)
 data := make([]byte,10)
 totalData := make([]byte,0,100)

 for {
  // 设置偏移量 
  file.Seek(offset,0)
  offset += 10
  // 读取数据
  _,error = file.Read(data)
   if error != nil{
     fmt.Println(error)
     break
   }
  // 拼接数据
  totalData = append(totalData,data...)
   fmt.Println(string(data))
 }
  fmt.Println(string(totalData))
}
```
`file.Seek(offset,0) offset = 0` 从文件中读取10个数据,之后偏移量设置为offset = 10,则从文件内容第11个字节开始读取，当Read方法读取文件到结尾时,会返回EOF标识,这个时候程序退出for循环  

### 修改文件权限  
```go
func Lchown(name string, uid, gid int) error
```

### 管道的用法  
```go
func Pipe() (r *File, w *File, err error)
```
这个方法主要在协程之间进行数据传递，r.read 方法会等待接受w文件中写数据  

```go
func main() {
  r,w,error := os.Pipe()
  go write(w)
  data := make([]byte,1000)
  // 如果数据没有写入w中,则此方法一直在等待
  n,_ := r.Read(data)
  if error != nil{
    println(error)
  }
  fmt.Println("读取数据：")
  fmt.Println(string(data[:n]))
}

func write(w *os.File){
  time.AfterFunc(time.Second* 2, func() {
     w.WriteString("ABCD")
     w.WriteString("EFGH")
     fmt.Println("数据已写入w")
  })
}
```

### 创建系统错误  
```go
func NewSyscallError(syscall string, err error) error
```

### 通过pid查找进行进程  
```go
func FindProcess(pid int) (*Process, error)
```
查找增加运行的进程,但是在 Unix 系统上，无论过程是否存在，FindProcess 都会成功并为给定的 PID 返回一个 Process  

### 杀死进程  
```go
    func (p *Process) Kill() error
```
```go
func main() {

 pid := os.Getpid()
 process,error := os.FindProcess(pid)
 if error != nil{
   fmt.Println(error)
 }
 process.Kill()
 // 进行杀死 下面就不会执行了
 fmt.Println(pid)
 fmt.Println(process)
}
```

### 释放与进程p关联的任何资源  
```go
func (p *Process) Release() error
```
Release释放进程p绑定的所有资源， 使它们（资源）不能再被（进程p）使用。只有没有调用Wait方法时才需要调用本方法  
```go
func (p *Process) Signal(sig Signal) error
func (p Process) Wait() (ProcessState, error)
func (p *ProcessState) Exited() bool
func (p *ProcessState) Pid() int
func (p *ProcessState) String() string
func (p *ProcessState) Success() bool
func (p *ProcessState) Sys() interface{}
func (p *ProcessState) SysUsage() interface{}
func (p *ProcessState) SystemTime() time.Duration
func (p *ProcessState) UserTime() time.Duration
func (e *SyscallError) Error() string
```

---
# `sort` 包  
sort包中实现了３种基本的排序算法：插入排序．快排和堆排序．和其他语言中一样，这三种方式都是不公开的，他们只在sort包内部使用．所以用户在使用sort包进行排序时无需考虑使用那种排序方式，sort.Interface定义的三个方法：获取数据集合长度的Len()方法、比较两个元素大小的Less()方法和交换两个元素位置的Swap()方法，就可以顺利对数据集合进行排序。sort包会根据实际数据自动选择高效的排序算法。  

## type Interface  
```go
type Interface interface {

    Len() int    // Len 为集合内元素的总数
  
    Less(i, j int) bool　//如果index为i的元素小于index为j的元素，则返回true，否则返回false

    Swap(i, j int)  // Swap 交换索引为 i 和 j 的元素
}
```
任何实现了 sort.Interface 的类型（一般为集合），均可使用该包中的方法进行排序。这些方法要求集合内列出元素的索引为整数。  

golang自身实现的interface有三种，Float64Slice，IntSlice，StringSlice  

```go

package main
 
import (
    "fmt"
    "sort"
)
 
//定义interface{},并实现sort.Interface接口的三个方法
type IntSlice []int
 
func (c IntSlice) Len() int {
    return len(c)
}
func (c IntSlice) Swap(i, j int) {
    c[i], c[j] = c[j], c[i]
}
func (c IntSlice) Less(i, j int) bool {
    return c[i] < c[j]
}
func main() {
    a := IntSlice{1, 3, 5, 7, 2}
    b := []float64{1.1, 2.3, 5.3, 3.4}
    c := []int{1, 3, 5, 4, 2}
    fmt.Println(sort.IsSorted(a)) //false
    if !sort.IsSorted(a) {
        sort.Sort(a) 
    }
    if !sort.Float64sAreSorted(b) {
        sort.Float64s(b)
    }
    if !sort.IntsAreSorted(c) {
        sort.Ints(c)
    }
    fmt.Println(a)//[1 2 3 5 7]
    fmt.Println(b)//[1.1 2.3 3.4 5.3]
    fmt.Println(c)// [1 2 3 4 5]
}
```

## Search 二分法查找  
`func Search(n int, f func(int) bool) int `  

使用二分法查找（常用于一个已排序、可索引的数据结构，如数组、切片），查找范围是[0:n],返回能使f(i)=true的最小i（0 <= i < n)，并且会假定，如果f(i)=true，则f(i+1)=true，即对于切片[0:n]，i之前的切片元素会使f()函数返回false，i及i之后的元素会使f()函数返回true。但是，当在切片中无法找到时f(i)=true的i时（此时切片元素都不能使f()函数返回true），Search()方法会返回n（而不是返回-1）。  

为了查找某个值，而不是某一范围的值时，如果slice以升序排序，则　f func中应该使用＞＝,如果slice以降序排序，则应该使用<=。  

```go
package main
 
import (
    "fmt"
    "sort"
)
 
func main() {
    a := []int{1, 2, 3, 4, 5}
    b := sort.Search(len(a), func(i int) bool { return a[i] >= 30 })
    fmt.Println(b)　　　　　　　//5，查找不到，返回a slice的长度５，而不是-1
    c := sort.Search(len(a), func(i int) bool { return a[i] <= 3 })
    fmt.Println(c)                             //0，利用二分法进行查找，返回符合条件的最左边数值的index，即为０
    d := sort.Search(len(a), func(i int) bool { return a[i] == 3 })
    fmt.Println(d)                          //2　　　
}
```
```go
func SearchFloat64s(a []float64, x float64) int　　
//SearchFloat64s 在float64s切片中搜索x并返回索引如Search函数所述. 返回可以插入x值的索引位置，如果x不存在，返回数组a的长度切片必须以升序排列
func SearchInts(a []int, x int) int  
//SearchInts 在ints切片中搜索x并返回索引如Search函数所述. 返回可以插入x值的索引位置，如果x不存在，返回数组a的长度切片必须以升序排列
func SearchStrings(a []string, x string) int
//SearchFloat64s 在strings切片中搜索x并返回索引如Search函数所述. 返回可以插入x值的索引位置，如果x不存在，返回数组a的长度切片必须以升序排列
```
其中需要注意的是，以上三种search查找方法，其对应的slice必须按照**升序**进行排序，否则会出现奇怪的结果．

## Stable 排序
`func Stable(data Interface)` ：Stable对data进行排序，不过排序过程中，如果data中存在相等的元素，则他们原来的顺序不会改变，即如果有两个相等元素num,他们的初始index分别为i和j，并且i < j，则利用Stable对data进行排序后，i依然小于ｊ．直接利用sort进行排序则不能够保证这一点．

## Reverse 逆序排序  
```go
func Reverse(data Interface) Interface

// eg.
package main
 
import (
    "fmt"
    "sort"
)
 
func main() {
    a := []int{1, 2, 5, 3, 4}
    fmt.Println(a)        // [1 2 5 3 4]
    sort.Sort(sort.Reverse(sort.IntSlice(a)))
    fmt.Println(a)        // [5 4 3 2 1]
}
```
---

# `strconv` 包  
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

# `strings` 包  
Go 中使用 `strings` 包对字符串进行操作。[中文文档](http://docscn.studygolang.com/pkg/strings/)、 [官方文档](http://golang.org/pkg/strings/) 或 [国内访问文档](http://docs.studygolang.com/pkg/strings/)  

## 前、后缀  
`HasPrefix` 判断字符串 `s` 是否以 `prefix` 开头：  
```go
    strings.HasPrefix(s, prefix string) bool
```
`HasSuffix` 判断字符串 `s` 是否以 `suffix` 结尾：  
```go
    strings.HasSuffix(s, suffix string) bool
```
## 字符串包含关系  
`Contains` 判断字符串 `s` 是否包含 `substr` :  
```go
    strings.Contains(s, substr string) bool
```

`func ContainsAny(s, chars string) bool` : 判断字符串s是否包含chars中任意一个字符，若chars为空，返回false  

`func ContainsRune(s string, r rune) bool` : 是否包含r rune  

## 索引字符串位置  
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
## 字符串替换  
`Replace` 用于将字符串 `str` 中的前 `n` 个字符串 `old` 替换为字符串 `new`，并范围一个新的字符串，如果 `n = -1` 啧替换所有有 `old` 为 `new`：  
```go
    strings.Replace(str, old, new, n) string
```
返回的是str的副本  

## 统计字符串出现的次数
`Count` 用于统计字符串 `str` 中字符串 `s` 出现的非重叠次数：  
```go
    strings.Count(s, str string) int
```
## 重复字符串  
`Repeat` 用于重复 `count` 次字符串 `s` 并返回一个新的字符串：
```go
    strings.Repeat(s, count int) string
```
## 修改字符串大小写  
`ToLower` 将字符串中的 Unicode 字符全部转换为相应的小写字符：
```go
    strings.Tolower(s) string
```
`ToUpper` 将字符串中的 Unicode 字符全部转换为相应的大写字符：
```go
    strings.ToUpper(s) string
```
`ToTitle` s 中的所有字符修改为其 Title 格式：  
```go
func ToTitle(s string) string
```

## 修剪字符串  
`strings.TrimSpace(s)` 用于剔除字符串开头和结尾的空白符号；  
`strings.Trim(s, "cut")` 用于剔除字符串开头和结尾的 `cut`；  
`TrimLeft` 或者 `TrimRight` 用于只剔除开头或结尾的字符串。  
```go
func TrimSpace(s string) string
func Trim(s string, cutset string) string
func TrimRight(s string, cutset string) string
```
注：TrimSuffix只是去掉s字符串结尾的suffix字符串，只是去掉１次，而TrimRight是一直去掉s字符串右边的字符串，只要有响应的字符串就去掉，是一个多次的过程，这也是二者的本质区别．

## 分割字符串  
`strings.Fields(s)` 会利用 1 个或多个空白符号(\t, \n, \v, \f, \r, ' ', U+0085 (NEL), U+00A0 (NBSP))来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的空 slice。  

`strings.Split(s, sep)` 用于自定义分割符号来对指定字符串进行分割，同样范围 slice。  
因为这 2 个函数都会返回 slice，所以习惯使用 for - range 循环来对其进行处理。  
## 拼接 slice 到字符串  
`Join` 用于将元素类型为 string 的 slice 使用分割符号来拼接组成一个字符串：  
```go
    strings.Join(sl [string], sep string) string
```
## 从字符串中读取内容  
`strings.NewReader(str)` 用于生成一个 'Reader' 并读取字符串中的内容，然后返回指向该 `Reader` 的指针，从其他类型读取内容的函数还有：  
`Read` 从 []byte 中读取内容；  
`ReadByte()` 和 `ReadRune()` 从字符串中读取下一个 byte 或者 rune。  

## 比较两字符串字典顺序  
`func Compare(a, b string) int`  
return 0 if a==b, -1 if a < b, and +1 if a > b.  
但不常用，通常用 go 自身的 ==, >, < 来比较，更快更易懂。  

## 忽略大小写判断s和t是否相等  
`func EqualFold(s, t string) bool`

## Map  
`func Map(mapping func(rune) rune, s string) string` ：s 中满足 mapping(rune) 的字符替换为 mapping(rune) 的返回值。如果 mapping(rune) 返回负数，则相应的字符将被删除。  
```go
func Slash(r rune) rune {
    if r == '\\' {
        return '/'
    }
    return r
}
func main() {
 
    s := "C:\\Windows\\System32\\FileName"
    ms := strings.Map(Slash, s)
    fmt.Printf("%q\n", ms) // "C:/Windows/System32/FileName"
}
```

## reader.go  
### Reader结构
```go
// Reader 结构通过读取字符串，实现了 io.Reader，io.ReaderAt，
// io.Seeker，io.WriterTo，io.ByteScanner，io.RuneScanner 接口
type Reader struct {
    s string // 要读取的字符串
    i int // 当前读取的索引位置，从 i 处开始读取数据
    prevRune int // 读取的前一个字符的索引位置，小于 0 表示之前未读取字符
}

// 通过字符串 s 创建 strings.Reader 对象
// 这个函数类似于 bytes.NewBufferString
// 但比 bytes.NewBufferString 更有效率，而且只读
func NewReader(s string) *Reader { return &Reader{s, 0, -1} }
```
### Len 返回 r.i 之后的所有数据的字节长度  
```go
func (r *Reader) Len() int
```

### Read 将 r.i 之后的所有数据写入到 b 中（如果 b 的容量足够大）
返回读取的字节数和读取过程中遇到的错误  
如果无可读数据，则返回 io.EOF  
```go
func (r *Reader) Read(b []byte) (n int, err error)

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 创建长度为 5 个字节的缓冲区
    b := make([]byte, 5)
    // 循环读取 r 中的字符串
    for n, _ := r.Read(b); n > 0; n, _ = r.Read(b) {
        fmt.Printf("%q, ", b[:n]) // "Hello", " Worl", "d!"
    }
}
```
### ReadAt 将 off 之后的所有数据写入到 b 中（如果 b 的容量足够大）  
返回读取的字节数和读取过程中遇到的错误  
如果无可读数据，则返回 io.EOF  
如果数据被一次性读取完毕，则返回 io.EOF  
```go
func (r *Reader) ReadAt(b []byte, off int64) (n int, err error) 

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 创建长度为 5 个字节的缓冲区
    b := make([]byte, 5)
    // 读取 r 中指定位置的字符串
    n, _ := r.ReadAt(b, 0)
    fmt.Printf("%q\n", b[:n]) // "Hello"
    // 读取 r 中指定位置的字符串
    n, _ = r.ReadAt(b, 6)
    fmt.Printf("%q\n", b[:n]) // "World"
}
```

### ReadByte 将 r.i 之后的一个字节写入到返回值 b 中  
返回读取的字节和读取过程中遇到的错误  
如果无可读数据，则返回 io.EOF  
```go
func (r *Reader) ReadByte() (b byte, err error)

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 读取 r 中的一个字节
    for i := 0; i < 3; i++ {
        b, _ := r.ReadByte()
        fmt.Printf("%q, ", b) // 'H', 'e', 'l',
    }
}
```
### UnreadByte 撤消前一次的 ReadByte 操作，即 r.i--
```go
func (r *Reader) UnreadByte() error

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 读取 r 中的一个字节
    for i := 0; i < 3; i++ {
        b, _ := r.ReadByte()
        fmt.Printf("%q, ", b) // 'H', 'H', 'H',
        r.UnreadByte()        // 撤消前一次的字节读取操作
    }
}
```

### ReadRune 将 r.i 之后的一个字符写入到返回值 ch 中 
ch： 读取的字符  
size：ch 的编码长度  
err： 读取过程中遇到的错误  
如果无可读数据，则返回 io.EOF  
如果 r.i 之后不是一个合法的 UTF-8 字符编码，则返回 utf8.RuneError 字符  
```go
func (r *Reader) ReadRune() (ch rune, size int, err error) 

func main() {
    s := "你好 世界！"
    // 创建 Reader
    r := strings.NewReader(s)
    // 读取 r 中的一个字符
    for i := 0; i < 5; i++ {
        b, n, _ := r.ReadRune()
        fmt.Printf(`"%c:%v", `, b, n)
        // "你:3", "好:3", " :1", "世:3", "界:3",
    }
}
```

### 撤消前一次的 ReadRune 操作  
```go
func (r *Reader) UnreadRune() error

func main() {
    s := "你好 世界！"
    // 创建 Reader
    r := strings.NewReader(s)
    // 读取 r 中的一个字符
    for i := 0; i < 5; i++ {
        b, _, _ := r.ReadRune()
        fmt.Printf("%q, ", b)
        // '你', '你', '你', '你', '你',
        r.UnreadRune() // 撤消前一次的字符读取操作
    }
}
```

### Seek 用来移动 r 中的索引位置  
offset：要移动的偏移量，负数表示反向移动  
whence：从那里开始移动，0：起始位置，1：当前位置，2：结尾位置  
如果 whence 不是 0、1、2，则返回错误信息  
如果目标索引位置超出字符串范围，则返回错误信息  
目标索引位置不能超出 1 << 31，否则返回错误信息  
```go
func (r *Reader) Seek(offset int64, whence int) (int64, error)

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 创建读取缓冲区
    b := make([]byte, 5)
    // 读取 r 中指定位置的内容
    r.Seek(6, 0) // 移动索引位置到第 7 个字节
    r.Read(b)    // 开始读取
    fmt.Printf("%q\n", b)
    r.Seek(-5, 1) // 将索引位置移回去
    r.Read(b)     // 继续读取
    fmt.Printf("%q\n", b)
}
```

### WriteTo 将 r.i 之后的数据写入接口 w 中  
```go
func (r *Reader) WriteTo(w io.Writer) (n int64, err error)

func main() {
    s := "Hello World!"
    // 创建 Reader
    r := strings.NewReader(s)
    // 创建 bytes.Buffer 对象，它实现了 io.Writer 接口
    buf := bytes.NewBuffer(nil)
    // 将 r 中的数据写入 buf 中
    r.WriteTo(buf)
    fmt.Printf("%q\n", buf) // "Hello World!"
}
```

## replace.go  
### Replacer 根据一个替换列表执行替换操作  
```go
type Replacer struct {
    Replace(s string) string
    WriteString(w io.Writer, s string) (n int, err error)
}
```

### NewReplacer 通过“替换列表”创建一个 Replacer 对象  
按照“替换列表”中的顺序进行替换，只替换非重叠部分。  
如果参数的个数不是偶数，则抛出异常。  
如果在“替换列表”中有相同的“查找项”，则后面重复的“查找项”会被忽略  
```go
func NewReplacer(oldnew ...string) *Replacer
```
### Replace 返回对 s 进行“查找和替换”后的结果  
Replace 使用的是 Boyer-Moore 算法，速度很快
```go
func (r *Replacer) Replace(s string) string

func main() {
    srp := strings.NewReplacer("Hello", "你好", "World", "世界", "!", "！")
    s := "Hello World!Hello World!hello world!"
    rst := srp.Replace(s)
    fmt.Print(rst) // 你好 世界！你好 世界！hello world！
}
//注：这两种写法均可．
func main() {
 
    wl := []string{"Hello", "Hi", "Hello", "你好"}
    srp := strings.NewReplacer(wl...)
    s := "Hello World! Hello World! hello world!"
    rst := srp.Replace(s)
    fmt.Print(rst) // Hi World! Hi World! hello world!
```
### WriteString 对 s 进行“查找和替换”，然后将结果写入 w 中  
```go
func (r *Replacer) WriteString(w io.Writer, s string) (n int, err error)

func main() {
    wl := []string{"Hello", "你好", "World", "世界", "!", "！"}
    srp := strings.NewReplacer(wl...)
    s := "Hello World!Hello World!hello world!"
    srp.WriteString(os.Stdout, s)
    // 你好 世界！你好 世界！hello world！
```

---
# `time` 包：时间和日期
包中包括了两类时间：时间点（某一时刻）和时长（某一个时间段）    
`time` 包提供了一个数据类型 `time.Time`（作为值使用）以及显示和测量时间和日期的功能函数。  

戳→[中文文档](http://docscn.studygolang.com/pkg/time/)、 [官方文档](http://golang.org/pkg/time/) 、 [国内访问页面](http://docs.studygolang.com/pkg/time/)   

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

## 时间敞亮（时间格式化）  
```go
const (
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
)
```
这些常量是在time包中进行time 格式化 和time解析而预定义的一些常量，其实他们使用的都是一个特定的时间：`Mon Jan 2 15:04:05 MST 2006`  

## 函数  
### time 组成  
`time.Duration` 时间长度，消耗时间
`time.Time` 时间点  
`time.C` 放时间的通道(tine.C := make(chan time.Time))

### After 函数  
```go
func After(d Duration) <-chan Time
```
表示多少时间之后，但是在去除channel内容之前不阻塞，后续程序可以继续执行。
```go
func Sleep(d Duration)
```
表示休眠多少时间，休眠时处于阻塞状态，后续程序无法执行。  

鉴于After特性，其通常用来处理程序超时问题，如下所示：  
```go
select {
case m := <-c:
    handle(m)
case <-time.After(5 * time.Minute):
    fmt.Println("timed out")
}
```

```go
func Tick(d Duration) <-chan Time
```
time.Tick(time.Duration)用法和time.After差不多，但是它是表示每隔多少时间之后，是一个重复的过程，其他与After一致。  

```go
type Duration int64 
//时间长度，其对应的时间单位有Nanosecond，Microsecond,Millisecond,Second,Minute,Hour

func ParseDuration(s string) (Duration, error)
//传入字符串，返回响应的时间，其中传入的字符串中的有效时间单位如下：h,m,s,ms,us,ns，其他单位均无效，如果传入无效时间单位，则会返回０ 

func Since(t Time) Duration 
//表示自从t时刻以后过了多长时间，是一个时间段，相当于time.Now().Sub(t)

func (d Duration) Hours() float64 
//将制定时间段换算为float64类型的Hour为单位进行输出

func (d Duration) Minutes() float64 
//将制定时间段换算为float64类型的Minutes为单位进行输出

func (d Duration) Nanoseconds() int64 
//将制定时间段换算为int64类型的Nanoseconds为单位进行输出

func (d Duration) Seconds() float64 
//将制定时间段换算为float64类型的Seconds为单位进行输出

func(d Duration) String() string  
//与ParseDuration函数相反，该函数是将时间段转化为字符串输出
```

```go
type Location
func FixedZone(name string, offset int) *Location
func LoadLocation(name string) (*Location, error)
func (l *Location) String() string
```

```go
type Month // 定义了1年的12个月

func (m Month) String() string  //将时间月份以字符串形式打印出来．如fmt.Println(time.June.String())则打印出June
```

```go
type ParseError
func (e *ParseError) Error() string
```

```go
type Ticker  
//主要用来按照指定的时间周期来调用函数或者计算表达式，
//通常的使用方式是利用go新开一个协程使用，它是一个断续器

func NewTicker(d Duration) *Ticker
//新生成一个ticker,此Ticker包含一个channel，
//此channel以给定的duration发送时间。duration d必须大于0

func (t *Ticker) Stop()  
//用于关闭相应的Ticker，但并不关闭channel
```
例子：  
使用时间控制停止ticker：  
```go
ticker := time.NewTicker(time.Millisecond * 500)
    go func() {
        for t := range ticker.C {
            fmt.Println("Tick at", t)
        }
    }()

time.Sleep(time.Millisecond * 1500)   
//阻塞，则执行次数为sleep的休眠时间/ticker的时间
ticker.Stop()     
fmt.Println("Ticker stopped")
```
使用channel控制停止ticker：  
```go
ticker := time.NewTicker(time.Millisecond * 500)
c := make(chan int，num) //num为指定的执行次数
go func() {
    for t := range ticker.C {
              c<-1
               fmt.Println("Tick at", t)
                
    }
}()
        
ticker.Stop()
```
这种情况下，在执行num次以Ticker时间为单位的函数之后，c　channel中已满，以后便不会再执行对应的函数  

```go
type Time  //包括日期和时间
func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time　
//按照指定格式输入数据后，便会按照如下格式输出对应的时间，输出格式为
//yyyy-mm-dd hh:mm:ss + nsec nanoseconds，　其中loc必须指定，否则便会panic

//eg，
t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
fmt.Printf("Go launched at %s\n", t.Local())
//输出为：
Go launched at 2009-11-10 15:00:00 -0800 PST
```

```go
func Now() Time //返回当前时间，包括日期，时间和时区

func Parse(layout, value string) (Time, error)　//输入格式化layout和时间字符串，输出时间


func ParseInLocation(layout, value string, loc *Location) (Time, error)


func Unix(sec int64, nsec int64) Time　//返回本地时间


func (t Time) Add(d Duration) Time　　//增加时间


func (t Time) AddDate(years int, months int, days int) Time//增加日期


func (t Time) After(u Time) bool　　//判断时间t是否在时间ｕ的后面


func (t Time) Before(u Time) bool　//判断时间t是否在时间ｕ的前面



func (t Time) Clock() (hour, min, sec int)　//获取时间ｔ的hour,min和second


func (t Time) Date() (year int, month Month, day int)　//获取时间ｔ的year,month和day


func (t Time) Day() int   //获取时间ｔ的day


func (t Time) Equal(u Time) bool  //判断时间t和时间u是否相同



func (t Time) Format(layout string) string  //时间字符串格式化


func (t *Time) GobDecode(data []byte) error //编码为god



func (t Time) GobEncode() ([]byte, error)//解码god


func (t Time) Hour() int　//获取时间ｔ的小时


func (t Time) ISOWeek() (year, week int)//获取时间ｔ的年份和星期


func (t Time) In(loc *Location) Time//获取loc时区的时间ｔ的对应时间


func (t Time) IsZero() bool　//判断是否为０时间实例January 1, year 1, 00:00:00 UTC


func (t Time) Local() Time　//获取当地时间


func (t Time) Location() *Location   //获取当地时区


func (t Time) MarshalBinary() ([]byte, error)　//marshal binary序列化，将时间t序列化后存入[]byte数组中


func (t Time) MarshalJSON() ([]byte, error)     //marshal json序列化，将时间t序列化后存入[]byte数组中


func (t Time) MarshalText() ([]byte, error)    //marshal text序列化，将时间t序列化后存入[]byte数组中
```
例子：
```go
t := time.Date(0, 0, 0, 12, 15, 30, 918273645, time.UTC)
round := []time.Duration{
    time.Nanosecond,
    time.Microsecond,
    time.Millisecond,
    time.Second,
    2 * time.Second,
    time.Minute,
    10 * time.Minute,
    time.Hour,
}

for _, d := range round {
    fmt.Printf("t.Round(%6s) = %s\n", d, t.Round(d).Format("15:04:05.999999999"))
}
```
```go
func (t Time) Second() int　//获取时间ｔ的秒

func (t Time) String() string　//获取时间ｔ的字符串表示


func (t Time) Sub(u Time) Duration　//与Add相反，Sub表示从时间ｔ中减去时间ｕ


func (t Time) Truncate(d Duration) Time　//去尾法求近似值
```
例子：  
```go
t, _ := time.Parse("2006 Jan 02 15:04:05", "2012 Dec 07 12:15:30.918273645")
trunc := []time.Duration{
    time.Nanosecond,
    time.Microsecond,
    time.Millisecond,
    time.Second,
    2 * time.Second,
    time.Minute,
    10 * time.Minute,
    time.Hour,
}

for _, d := range trunc {
    fmt.Printf("t.Truncate(%6s) = %s\n", d, t.Truncate(d).Format("15:04:05.999999999"))
}

输出：

t.Truncate(   1ns) = 12:15:30.918273645
t.Truncate(   1µs) = 12:15:30.918273
t.Truncate(   1ms) = 12:15:30.918
t.Truncate(    1s) = 12:15:30
t.Truncate(    2s) = 12:15:30
t.Truncate(  1m0s) = 12:15:00
t.Truncate( 10m0s) = 12:10:00
t.Truncate(1h0m0s) = 12:00:00
```

```go
func (t Time) UTC() Time　//将本地时间变换为UTC时区的时间并返回


func (t Time) Unix() int64　//返回Unix时间，该时间是从January 1, 1970 UTC这个时间开始算起的．


func (t Time) UnixNano() int64　//以纳秒为单位返回Unix时间


func (t *Time) UnmarshalBinary(data []byte) error　//将data数据反序列化到时间ｔ中


func (t *Time) UnmarshalJSON(data []byte) (err error)　//将data数据反序列化到时间ｔ中


func (t *Time) UnmarshalText(data []byte) (err error)　//将data数据反序列化到时间ｔ中


func (t Time) Weekday() Weekday　//获取时间ｔ的Weekday


func (t Time) Year() int　　　//获取时间ｔ的Year


func (t Time) YearDay() int     //获取时间ｔ的YearDay，即１年中的第几天


func (t Time) Zone() (name string, offset int)
```
```go
type Timer　//用于在指定的Duration类型时间后调用函数或计算表达式，它是一个计时器


func AfterFunc(d Duration, f func()) *Timer　//和After差不多，意思是多少时间之后执行函数f


func NewTimer(d Duration) *Timer　//使用NewTimer(),可以返回的Timer类型在计时器到期之前,取消该计时器


func (t *Timer) Reset(d Duration) bool　//重新设定timer t的Duration d.


func (t *Timer) Stop() bool　//阻止timer事件发生，当该函数执行后，timer计时器停止，相应的事件不再执行
```
```go
type Weekday
func (d Weekday) String() string　//获取一周的字符串
```

### 一些例子：  
获取年月日方法  
```go
    t := time.Now()
    y := t.Year() // 年
    m := int(t.Month()) //月
    d := t.Day()// 日
    h := t.Hour() //小时
    min := t.Minute()//分钟
    s := t.Second()//秒
    fmt.Printf("%d-%d-%d",y,m,d)// 2017-12-26
    fmt.Printf("%d%02d%02d",y,m,d) // 20180101

```

获取格式为:2017-12-29 16:58:39  
```go
    t := time.Now()
    fmt.Println(t.String()[:19])
    layout := "2006-01-02 15:04:05"
    fmt.Println(t.Format(layout))
```

获取三个小时之前的时间  
```go
    du, _ := time.ParseDuration("-3h")
    t := time.Now().Add(du)
```

获取三天之前的时间
```go
    t := time.Now()
    t.AddDate(0,0,-3) 
    fmt.Println(t)
```

t.AddDate(year,month,day) 正数向前，负数向后  

时间戳转time.Time 类型  
```go
    timestamp := 1513580645
    t := time.Unix(timestamp,0)
```

获取时间戳：  
```go
    t := time.Now()
    timestamp := t.Unix()
```

两个时间的间隔  
```go
    t1 := time.Now()
    //do some function
    t2 := time.Now()
    sub := t2.Since(t1)
```

字符串转时间  
```go
    layout := "2006-01-02 15:04:05" 
    // 这个格式，不能改，据说是Go诞生的时间，可能有特殊含义吧
    t , err := time.Parse(layout,"2017-12-26 17:20:21")
    fmt.Println(t)
```

标准时间字符串转化为时间  
使用byte读取数据库当中的timestamp字段时，会出现标准的时间格式  
```go
    layout := "2006-01-02T15:04:05Z07:00"
    timeStr := "2018-01-02T11:30:21Z"
    t , _ := time.Parse(layout,timeStr)
    // t , _ := time.Parse(time.RFC3339,timeStr)
    fmt.Println(t)
```

---
*`to be continued...`*  

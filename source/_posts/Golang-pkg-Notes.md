---
title: Golang pkg Notes
date: 2019-03-20 22:16:19
tags: Go
categories: Go
permalink: Golang-pkg
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
# `Strconv` 包  
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
# `time` 包：时间和日期  
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
*`to be continued...`*  

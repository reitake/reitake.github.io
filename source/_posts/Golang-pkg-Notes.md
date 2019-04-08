---
title: Golang pkg Notes
date: 2019-03-20 22:16:19
tags: Go
categories: Go
permalink: Golang-pkg
---
<center> <font color="#bababa">***Go 语言常用包笔记***</font><br/> </center>
<!--more-->
---

# `bufio` 包  

实现了带缓存的 I/O 操作。  

## Read 相关函数  

### NewReaderSize、NewReader  
```go
func NewReaderSize(rd io.Reader, size int) *Reader
```

NewReaderSize 将 rd 封装成一个带缓存的 bufio.Reader 对象，缓存大小由 size 指定（如果小于 16 则会被设置为 16）。  

如果 rd 的基类型就是有足够缓存的 bufio.Reader 类型，则直接将 rd 转换为基类型返回。  

```go
func NewReader(rd io.Reader) *Reader
```

NewReader 相当于 NewReaderSize(rd, 4096)。  

bufio.Reader 实现了如下接口：  

```go
io.Reader
io.WriterTo
io.ByteScanner
io.RuneScanner
```

### Peek  
```go
func (b *Reader) Peek(n int) ([]byte, error)
```

Peek 返回缓存的一个切片，该切片引用缓存中前 n 个字节的数据，该操作不会将数据读出，只是引用，引用的数据在下一次读取操作之前是有效的。如果切片长度小于 n，则返回一个错误信息说明原因。如果 n 大于缓存的总大小，则返回 ErrBufferFull。  

### Read  
```go
func (b *Reader) Read(p []byte) (n int, err error)
```

Read 从 b 中读出数据到 p 中，返回读出的字节数和遇到的错误。  

如果缓存不为空，则只能读出缓存中的数据，不会从底层 io.Reader中提取数据，如果缓存为空，则：  

1、len(p) >= 缓存大小，则跳过缓存，直接从底层 io.Reader 中读出到 p 中。  

2、len(p) < 缓存大小，则先将数据从底层 io.Reader 中读取到缓存中，再从缓存读取到 p 中。  

### Buffered  
```go
func (b *Reader) Buffered() int
```

Buffered 返回缓存中未读取的数据的长度。  

### Discard  
```go
func (b *Reader) Discard(n int) (discarded int, err error)
```

Discard 跳过后续的 n 个字节的数据，返回跳过的字节数。  

如果结果小于 n，将返回错误信息。  

如果 n 小于缓存中的数据长度，则不会从底层提取数据。  

### ReadSlice  
```go
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```

ReadSlice 在 b 中查找 delim 并返回 delim 及其之前的所有数据。  

该操作会读出数据，返回的切片是已读出的数据的引用，切片中的数据在下一次读取操作之前是有效的。  

如果找到 delim，则返回查找结果，err 返回 nil。  

如果未找到 delim，则：  

1、缓存不满，则将缓存填满后再次查找。  

2、缓存是满的，则返回整个缓存，err 返回 ErrBufferFull。  

如果未找到 delim 且遇到错误（通常是 io.EOF），则返回缓存中的所有数据和遇到的错误。  
因为返回的数据有可能被下一次的读写操作修改，所以大多数操作**应该使用 ReadBytes 或 ReadString，它们返回的是数据的拷贝**。  

### ReadLine  
```go
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)

// ReadLine 是一个低水平的行读取原语，大多数情况下，应该使用
// ReadBytes('\n') 或 ReadString('\n')，或者使用一个 Scanner。
//
// ReadLine 通过调用 ReadSlice 方法实现，返回的也是缓存的切片。用于
// 读取一行数据，不包括行尾标记（\n 或 \r\n）。
//
// 只要能读出数据，err 就为 nil。如果没有数据可读，则 isPrefix 返回
// false，err 返回 io.EOF。
//
// 如果找到行尾标记，则返回查找结果，isPrefix 返回 false。
// 如果未找到行尾标记，则：
// 1、缓存不满，则将缓存填满后再次查找。
// 2、缓存是满的，则返回整个缓存，isPrefix 返回 true。
//
// 整个数据尾部“有一个换行标记”和“没有换行标记”的读取结果是一样。
//
// 如果 ReadLine 读取到换行标记，则调用 UnreadByte 撤销的是换行标记，
// 而不是返回的数据。
```

### ReadBytes、ReadString  
```go
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
```

ReadBytes 功能同 ReadSlice，只不过返回的是缓存的拷贝。  

```go
func (b *Reader) ReadString(delim byte) (line string, err error)
```

ReadString 功能同 ReadBytes，只不过返回的是字符串。  

### Reset  
```go
func (b *Reader) Reset(r io.Reader)
```

Reset 将 b 的底层 Reader 重新指定为 r，同时丢弃缓存中的所有数据，复位
所有标记和错误信息。 bufio.Reader。  

### 示例1：Peek、Read、Discard、Buffered  
```go
func main() {
    sr := strings.NewReader("ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890")
    buf := bufio.NewReaderSize(sr, 0)
    b := make([]byte, 10)

    fmt.Println(buf.Buffered()) // 0
    s, _ := buf.Peek(5)
    s[0], s[1], s[2] = 'a', 'b', 'c'
    fmt.Printf("%d   %q\n", buf.Buffered(), s) // 16   "abcDE"

    buf.Discard(1)

    for n, err := 0, error(nil); err == nil; {
        n, err = buf.Read(b)
        fmt.Printf("%d   %q   %v\n", buf.Buffered(), b[:n], err)
    }
    // 5   "bcDEFGHIJK"   <nil>
    // 0   "LMNOP"   <nil>
    // 6   "QRSTUVWXYZ"   <nil>
    // 0   "123456"   <nil>
    // 0   "7890"   <nil>
    // 0   ""   EOF
}
```

### 示例2：ReadLine  
```go
func main() {
    sr := strings.NewReader("ABCDEFGHIJKLMNOPQRSTUVWXYZ\n1234567890")
    buf := bufio.NewReaderSize(sr, 0)

    for line, isPrefix, err := []byte{0}, false, error(nil); len(line) > 0 && err == nil; {
        line, isPrefix, err = buf.ReadLine()
        fmt.Printf("%q   %t   %v\n", line, isPrefix, err)
    }
    // "ABCDEFGHIJKLMNOP"   true   <nil>
    // "QRSTUVWXYZ"   false   <nil>
    // "1234567890"   false   <nil>
    // ""   false   EOF

    fmt.Println("----------")

    // 尾部有一个换行标记
    buf = bufio.NewReaderSize(strings.NewReader("ABCDEFG\n"), 0)

    for line, isPrefix, err := []byte{0}, false, error(nil); len(line) > 0 && err == nil; {
        line, isPrefix, err = buf.ReadLine()
        fmt.Printf("%q   %t   %v\n", line, isPrefix, err)
    }
    // "ABCDEFG"   false   <nil>
    // ""   false   EOF

    fmt.Println("----------")

    // 尾部没有换行标记
    buf = bufio.NewReaderSize(strings.NewReader("ABCDEFG"), 0)

    for line, isPrefix, err := []byte{0}, false, error(nil); len(line) > 0 && err == nil; {
        line, isPrefix, err = buf.ReadLine()
        fmt.Printf("%q   %t   %v\n", line, isPrefix, err)
    }
    // "ABCDEFG"   false   <nil>
    // ""   false   EOF
}
```

### 示例3：ReadSlice  
```go
func main() {
    // 尾部有换行标记
    buf := bufio.NewReaderSize(strings.NewReader("ABCDEFG\n"), 0)

    for line, err := []byte{0}, error(nil); len(line) > 0 && err == nil; {
        line, err = buf.ReadSlice('\n')
        fmt.Printf("%q   %v\n", line, err)
    }
    // "ABCDEFG\n"   <nil>
    // ""   EOF

    fmt.Println("----------")

    // 尾部没有换行标记
    buf = bufio.NewReaderSize(strings.NewReader("ABCDEFG"), 0)

    for line, err := []byte{0}, error(nil); len(line) > 0 && err == nil; {
        line, err = buf.ReadSlice('\n')
        fmt.Printf("%q   %v\n", line, err)
    }
    // "ABCDEFG"   EOF
}
```

## Write 相关函数  

### NewWriterSize、NewWriter  
```go
func NewWriterSize(wr io.Writer, size int) *Writer
// NewWriterSize 将 wr 封装成一个带缓存的 bufio.Writer 对象，
// 缓存大小由 size 指定（如果小于 4096 则会被设置为 4096）。
// 如果 wr 的基类型就是有足够缓存的 bufio.Writer 类型，则直接将
// wr 转换为基类型返回。
```

```go
func NewWriter(wr io.Writer) *Writer
// NewWriter 相当于 NewWriterSize(wr, 4096)
```

bufio.Writer 实现了如下接口：  
```
io.Writer
io.ReaderFrom
io.ByteWriter
```

### WriteString  
```go
func (b *Writer) WriteString(s string) (int, error)
```
WriteString 功能同 Write，只不过写入的是字符串。  

### WriteRune  
```go
func (b *Writer) WriteRune(r rune) (size int, err error)
```
WriteRune 向 b 写入 r 的 UTF-8 编码，返回 r 的编码长度。  

### Flush  
```go
func (b *Writer) Flush() error
```
Flush 将缓存中的数据提交到底层的 io.Writer 中。

### Available  
```go
func (b *Writer) Available() int
```
Available 返回缓存中未使用的空间的长度。  

### Buffered  
```go
func (b *Writer) Buffered() int
```
Buffered 返回缓存中未提交的数据的长度。  

### Reset  
```go
func (b *Writer) Reset(w io.Writer)
```
Reset 将 b 的底层 Writer 重新指定为 w，同时丢弃缓存中的所有数据，复位所有标记和错误信息。相当于创建了一个新的 bufio.Writer。  

### 示例1：Available、Buffered、WriteString、Flush  
```go
func main() {
    buf := bufio.NewWriterSize(os.Stdout, 0)
    fmt.Println(buf.Available(), buf.Buffered()) // 4096 0

    buf.WriteString("ABCDEFGHIJKLMNOPQRSTUVWXYZ")
    fmt.Println(buf.Available(), buf.Buffered()) // 4070 26

    // 缓存后统一输出，避免终端频繁刷新，影响速度
    buf.Flush() // ABCDEFGHIJKLMNOPQRSTUVWXYZ
}
```

## type ReadWriter  
ReadWriter 集成了 bufio.Reader 和 bufio.Writer。  
```go
type ReadWriter struct {
    *Reader
    *Writer
}
```

```go
func NewReadWriter(r *Reader, w *Writer) *ReadWriter
```
NewReadWriter 将 r 和 w 封装成一个 bufio.ReadWriter 对象。  

## type Scanner
```go
type Scanner struct { ... }
// Scanner 提供了一个方便的接口来读取数据，例如遍历多行文本中的行。Scan 方法会通过
// 一个“匹配函数”读取数据中符合要求的部分，跳过不符合要求的部分。“匹配函数”由调
// 用者指定。本包中提供的匹配函数有“行匹配函数”、“字节匹配函数”、“字符匹配函数”
// 和“单词匹配函数”，用户也可以自定义“匹配函数”。默认的“匹配函数”为“行匹配函
// 数”，用于获取数据中的一行内容（不包括行尾标记）
//
// Scanner 使用了缓存，所以匹配部分的长度不能超出缓存的容量。默认缓存容量为 4096 -
// bufio.MaxScanTokenSize，用户可以通过 Buffer 方法指定自定义缓存及其最大容量。
//
// Scan 在遇到下面的情况时会终止扫描并返回 false（扫描一旦终止，将无法再继续）：
// 1、遇到 io.EOF
// 2、遇到读写错误
// 3、“匹配部分”的长度超过了缓存的长度
//
// 如果需要对错误进行更多的控制，或“匹配部分”超出缓存容量，或需要连续扫描，则应该
// 使用 bufio.Reader
```

```go
func NewScanner(r io.Reader) *Scanner
```
NewScanner 创建一个 Scanner 来扫描 r，默认匹配函数为 ScanLines。  

```go
func (s *Scanner) Buffer(buf []byte, max int)
```
Buffer 用于设置自定义缓存及其可扩展范围，如果 max 小于 len(buf)，则 buf 的尺寸将固定不可调。Buffer 必须在第一次 Scan 之前设置，否则会引发 panic。  

默认情况下，Scanner 会使用一个 4096 - bufio.MaxScanTokenSize 大小的内部缓存。  

```go
func (s *Scanner) Split(split SplitFunc)
```
Split 用于设置“匹配函数”，这个函数必须在调用 Scan 前执行。  

## type SplitFunc  
```go
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
// SplitFunc 用来定义“匹配函数”，data 是缓存中的数据。atEOF 标记数据是否读完。
// advance 返回 data 中已处理的数据的长度。token 返回找到的“匹配部分”，“匹配
// 部分”可以是缓存的切片，也可以是自己新建的数据（比如 bufio.errorRune）。“匹
// 配部分”将在 Scan 之后通过 Bytes 和 Text 反馈给用户。err 返回错误信息。
//
// 如果在 data 中无法找到一个完整的“匹配部分”则应返回 (0, nil, nil)，以便告诉
// Scanner 向缓存中填充更多数据，然后再次扫描（Scan 会自动重新扫描）。如果缓存已
// 经达到最大容量还没有找到，则 Scan 会终止并返回 false。
// 如果 data 为空，则“匹配函数”将不会被调用，意思是在“匹配函数”中不必考虑
// data 为空的情况。
//
// 如果 err != nil，扫描将终止，如果 err == ErrFinalToken，则 Scan 将返回 true，
// 表示扫描正常结束，如果 err 是其它错误，则 Scan 将返回 false，表示扫描出错。错误
// 信息可以在 Scan 之后通过 Err 方法获取。
//
// SplitFunc 的作用很简单，从 data 中找出你感兴趣的数据，然后返回，同时返回已经处理
// 的数据的长度。
```

```go
func (s *Scanner) Scan() bool
```
Scan 开始一次扫描过程，如果匹配成功，可以通过 Bytes() 或 Text() 方法取出结果，如果遇到错误，则终止扫描，并返回 false。  

```go
func (s *Scanner) Bytes() []byte
```
Bytes 将最后一次扫描出的“匹配部分”作为一个切片引用返回，下一次的 Scan 操作会覆盖本次引用的内容。  

```go
func (s *Scanner) Text() string
```
Text 将最后一次扫描出的“匹配部分”作为字符串返回（返回副本）。  

```go
func (s *Scanner) Err() error
```
Err 返回扫描过程中遇到的非 EOF 错误，供用户调用，以便获取错误信息。  

```go
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)
```
ScanBytes 是一个“匹配函数”用来找出 data 中的单个字节并返回。  

```go
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)
```
ScanRunes 是一个“匹配函数”，用来找出 data 中单个 UTF8 字符的编码。如果 UTF8 编码错误，则 token 会返回 "\xef\xbf\xbd"（即：U+FFFD），但只消耗 data 中的一个字节。  

这使得调用者无法区分“真正的U+FFFD字符”和“解码错误的返回值”。  

```go
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanLines 是一个“匹配函数”，用来找出 data 中的单行数据并返回（包括空行）。  

行尾标记可以是 \n 或 \r\n（返回值不包含行尾标记）  

```go
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanWords 是一个“匹配函数”，用来找出 data 中以空白字符分隔的单词。  

空白字符由 unicode.IsSpace 定义。  

### 示例1：扫描  
```go
func main() {
    // 逗号分隔的字符串，最后一项为空
    const input = "1,2,3,4,"
    scanner := bufio.NewScanner(strings.NewReader(input))
    // 定义匹配函数（查找逗号分隔的字符串）
    onComma := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
        for i := 0; i < len(data); i++ {
            if data[i] == ',' {
                return i + 1, data[:i], nil
            }
        }
        if atEOF {
            // 告诉 Scanner 扫描结束。
            return 0, data, bufio.ErrFinalToken
        } else {
            // 告诉 Scanner 没找到匹配项，让 Scan 填充缓存后再次扫描。
            return 0, nil, nil
        }
    }
    // 指定匹配函数
    scanner.Split(onComma)
    // 开始扫描
    for scanner.Scan() {
        fmt.Printf("%q ", scanner.Text())
    }
    // 检查是否因为遇到错误而结束
    if err := scanner.Err(); err != nil {
        fmt.Fprintln(os.Stderr, "reading input:", err)
    }
}
```

### 示例2：待检查扫描  
```go
func main() {
    const input = "1234 5678 1234567901234567890 90"
    scanner := bufio.NewScanner(strings.NewReader(input))
    // 自定义匹配函数
    split := func(data []byte, atEOF bool) (advance int, token []byte, err error) {
        // 获取一个单词
        advance, token, err = bufio.ScanWords(data, atEOF)
        // 判断其能否转换为整数，如果不能则返回错误
        if err == nil && token != nil {
            _, err = strconv.ParseInt(string(token), 10, 32)
        }
        // 这里包含了 return 0, nil, nil 的情况
        return
    }
    // 设置匹配函数
    scanner.Split(split)
    // 开始扫描
    for scanner.Scan() {
        fmt.Printf("%s\n", scanner.Text())
    }
    if err := scanner.Err(); err != nil {
        fmt.Printf("Invalid input: %s", err)
    }
}
```

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

# `errors` 包  

Go 语言使用 error 类型来返回函数执行过程中遇到的错误，如果返回的 error 值为 nil，则表示未遇到错误，否则 error 会返回一个字符串，用于说明遇到了什么错误。  

其实 error 只是一个接口，定义如下：  

```go
type error interface {
    Error() string
}
```
你可以用任何类型去实现它（只要添加一个 Error() 方法即可），也就是说，error 可以是任何类型，这意味着，函数返回的 error 值实际可以包含任意信息，不一定是字符串（当然字符串是必须的）。  

error 不一定表示一个错误，它可以表示任何信息，比如 io 包中就用 error 类型的 io.EOF 表示数据读取结束，而不是遇到了什么错误。再比如 path/filepath 包中用 error 类型的 filepath.SkipDir 表示跳过当前目录，继续遍历下一个目录，而不是遇到了什么错误。  

## `error.New()`函数
errors 包实现了一个最简单的 error 类型，只包含一个字符串，它可以记录大多数情况下遇到的错误信息。errors 包的用法也很简单，只有一个 New 函数，用于生成一个最简单的 error 对象：  

```go
// 将字符串 text 包装成一个 error 对象返回
func New(text string) error
```

eg：

```go
func SomeFunc() error {
    if 遇到错误 {
        return errors.New("遇到了某某错误")
    }
    return nil
}
```

## 自定义`error`类型  

如果你的程序需要记录更多的错误信息，比如时间、数值等信息，可以声明一个自定义的 error 类型。  

```go
package main

import (
    "fmt"
    "time"
)

type myError struct {
    err   string
    time  time.Time
    count int
}

func (m *myError) Error() string {
    return fmt.Sprintf("%s %d 次。时间：%v", m.err, m.count, m.time)
}

func newErr(s string, i int) *myError {
    return &myError{
        err:   s,
        time:  time.Now(),
        count: i,
    }
}

var count int

func SomeFunc() error {
    if true {
        count++
        return newErr("遇到某某情况", count)
    }
    return nil
}

func main() {
    for i := 0; i < 5; i++ {
        if err := SomeFunc(); err != nil {
            fmt.Println(err)
        }
    }
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

# `net/http` 包  

[文档](http://docscn.studygolang.com/pkg/net/http/)  

http包包含http客户端和服务端的实现,利用Get,Head,Post,以及PostForm实现HTTP或者HTTPS的请求。  

当客户端使用完response body后必须使用close对其进行关闭：  

```go
resp, err := http.Get("http://example.com/")
if err != nil {
    // handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```

## 变量  

```go
var (
        ErrHeaderTooLong        = &ProtocolError{"header too long"}
        ErrShortBody            = &ProtocolError{"entity body too short"}
        ErrNotSupported         = &ProtocolError{"feature not supported"}
        ErrUnexpectedTrailer    = &ProtocolError{"trailer header without chunked transfer encoding"}
        ErrMissingContentLength = &ProtocolError{"missing ContentLength in HEAD response"}
        ErrNotMultipart         = &ProtocolError{"request Content-Type isn't multipart/form-data"}
        ErrMissingBoundary      = &ProtocolError{"no multipart boundary param in Content-Type"}
)

var (
        ErrWriteAfterFlush = errors.New("Conn.Write called after Flush")
        ErrBodyNotAllowed  = errors.New("http: request method or response status code does not allow body")
        ErrHijacked        = errors.New("Conn has been hijacked")
        ErrContentLength   = errors.New("Conn.Write wrote more than the declared Content-Length")
)
var DefaultClient = &Client{} //默认客户端,被Get,Head以及Post使用
var DefaultServeMux = NewServeMux()//Serve使用的默认ServeMux
var ErrBodyReadAfterClose = errors.New("http: invalid Read on closed Body")//当读取一个request或者response body是在这个body已经关闭之后,便会返回该错误.这种错误主要发生情况是:当http handler调用writeheader或者write关于responsewrite之后进行读操作.
var ErrHandlerTimeout = errors.New("http: Handler timeout")//超时错误
var ErrLineTooLong = internal.ErrLineTooLong//当读取格式错误的分块编码时便会出现该错误.
var ErrMissingFile = errors.New("http: no such file")//当利用FormFile进行文件请求时,如果文件不存在或者请求中没有该文件就会出现该错误.
var ErrNoCookie = errors.New("http: named cookie not present")

var ErrNoLocation = errors.New("http: no Location header in response")
```

## 函数  

```go
func CanonicalHeaderKey(s string) string//返回header key的规范化形式,规范化形式是以"-"为分隔符,每一部分都是首字母大写,其他字母小写.例如"accept-encoding" 的标准化形式是 "Accept-Encoding".
func DetectContentType(data []byte) string//检查给定数据的内容类型Content-Type,最多检测512byte数据,如果有效的话,该函数返回一个MIME类型,否则的话,返回一个"application/octet-stream"

func Error(w ResponseWriter, error string, code int)//利用指定的错误信息和Http code来响应请求,其中错误信息必须是纯文本.
func NotFound(w ResponseWriter, r *Request)//返回HTTP404 not found错误

func Handle(pattern string, handler Handler)//将handler按照指定的格式注册到DefaultServeMux,ServeMux解释了模式匹配规则
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))//同上，主要用来实现动态文件内容的展示，这点与ServerFile()不同的地方。

func ListenAndServe(addr string, handler Handler) error//监听TCP网络地址addr然后调用具有handler的Serve去处理连接请求.通常情况下Handler是nil,使用默认的DefaultServeMux
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error//该函数与ListenAndServe功能基本相同,二者不同之处是该函数需要HTTPS连接.也就是说,必须给该服务Serve提供一个包含整数的秘钥的文件,如果证书是由证书机构签署的,那么证书文件必须是服务证书之后跟着CA证书.

func ServeFile(w ResponseWriter, r *Request, name string)//利用指定的文件或者目录的内容来响应相应的请求.
func SetCookie(w ResponseWriter, cookie *Cookie)//给w设定cookie
func StatusText(code int) string//对于http状态码返回文本表示,如果这个code未知,则返回空的字符串.
```

eg.  
```go
package main

import (
    "fmt"
    "net/http"
)

func Test(w http.ResponseWriter, r *http.Request) {
    // http.NotFound(w, r)//用于设置404问题
    // http.Error(w, "404 page not found", 404) //状态码需和描述相符

    http.ServeFile(w, r, "1.txt") //将1.txt中内容在w中显示.
    cookie := &http.Cookie{
        Name:  http.CanonicalHeaderKey("uid-test"), //Name值为Uid-Test
        Value: "1234",
    }
    r.AddCookie(cookie)
    fmt.Println(r.Cookie("uid-test")) //<nil> http: named cookie not present
    fmt.Println(r.Cookie("Uid-Test")) //Uid-Test=1234 <nil>
    fmt.Println(r.Cookies())          //[Uid-Test=1234]

}
func main() {

    stat := http.StatusText(200)
    fmt.Println(stat) //状态码200对应的状态OK

    stringtype := http.DetectContentType([]byte("test"))
    fmt.Println(stringtype) //text/plain; charset=utf-8

    http.HandleFunc("/test", Test)
    err := http.ListenAndServe(":9999", nil)
    if err != nil {
        fmt.Println(err)
    }

}
```

```go
func MaxBytesReader(w ResponseWriter, r io.ReadCloser, n int64) io.ReadCloser//该函数类似于io.LimitReader但是该函数是用来限制请求体的大小.与io.LimitReader不同的是,该函数返回一个ReaderCloser,当读超过限制时,返回一个non-EOF,并且当Close方法调用时,关闭底层的reader.该函数组织客户端恶意发送大量请求,浪费服务器资源.

func ParseHTTPVersion(vers string) (major, minor int, ok bool)//解析http字符串版本进行解析,"HTTP/1.0" 返回 (1, 0, true)//注解析的字符串必须以HTTP开始才能够正确解析，HTTP区分大小写，其他诸如http或者Http都不能够正确解析。
func ParseTime(text string) (t time.Time, err error)//解析时间头(例如data:header),解析格式如下面三种(HTTP/1.1中允许的):TimeFormat, time.RFC850, and time.ANSIC

func ProxyFromEnvironment(req *Request) (*url.URL, error)//该函数返回一个指定请求的代理URL,这是由环境变量HTTP_PROXY,HTTPS_PROXY以及NO_PROXY决定的,对于https请求,HTTPS_PROXY优先级高于HTTP_PROXY,环境值可能是一个URL或者一个host:port,其中这两种类型都是http调度允许的,如果不是这两种类型的数值便会返回错误.如果在环境中没有定义代理或者代理不应该用于该请求(定义为NO_PROXY),将会返回一个nil URL和一个nil error.作为一个特例,如果请求rul是"localhost",无论有没有port,那么将返回一个nil rul和一个nil error.
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)//返回一个用于传输的代理函数,该函数总是返回相同的URL
func Redirect(w ResponseWriter, r *Request, urlStr string, code int)//返回一个重定向的url给指定的请求,这个重定向url可能是一个相对请求路径的一个相对路径.
func Serve(l net.Listener, handler Handler) error//该函数接受listener l的传入http连接,对于每一个连接创建一个新的服务协程,这个服务协程读取请求然后调用handler来给他们响应.handler一般为nil,这样默认的DefaultServeMux被使用.
func ServeContent(w ResponseWriter, req *Request, name string, modtime time.Time, content io.ReadSeeker)//该函数使用提供的ReaderSeeker提供的内容来恢复请求,该函数相对于io.Copy的优点是可以处理范围类请求,设定MIME类型,并且处理了If-Modified-Since请求.如果未设定content-type类型,该函数首先通过文件扩展名来判断类型,如果失效的话,读取content的第一块数据并将他传递给DetectContentType进行类型判断.name可以不被使用,更进一步说,他可以为空并且不在respone中返回.如果modtime不是0时间,该时间则体现在response的最后一次修改的header中,如果请求包括一个If-Modified-Since header,该函数利用modtime来决定是否发送该content.该函数利用Seek功能来决定content的大小.
```

### **type Client**  
```go
type Client//Client是一个http客户端,默认客户端(DefaultClient)将使用默认的发送机制的客户端.Client的Transport字段一般会含有内部状态(缓存TCP连接),因此Client类型值应尽量被重用而不是创建新的。多个协程并发使用Clients是安全的.
type Client struct {
    // Transport指定执行独立、单次HTTP请求的机制如果Transport为nil,则使用DefaultTransport。
    Transport RoundTripper
    // CheckRedirect指定处理重定向的策略,如果CheckRedirect非nil,client将会在调用重定向之前调用它。
    // 参数req和via是将要执行的请求和已经执行的请求（时间越久的请求优先执行),如果CheckRedirect返回一个错误,
　　 //client的GetGet方法不会发送请求req,而是回之前得到的响应和该错误。
    // 如果CheckRedirect为nil，会采用默认策略：在连续10次请求后停止。
    CheckRedirect func(req *Request, via []*Request) error
    // Jar指定cookie管理器,如果Jar为nil,在请求中不会发送cookie,在回复中cookie也会被忽略。
    Jar CookieJar
    // Timeout指定Client请求的时间限制,该超时限制包括连接时间、重定向和读取response body时间。
    // 计时器会在Head,Get,Post或Do方法返回后开始计时并在读到response.body后停止计时。
　　// Timeout为零值表示不设置超时。
    // Client的Transport字段必须支持CancelRequest方法,否则Client会在尝试用Head,Get,Post或Do方法执行请求时返回错误。
    // Client的Transport字段默认值（DefaultTransport）支持CancelRequest方法。
    Timeout time.Duration
}
```

Client的方法：  

```go
func (c *Client) Do(req *Request) (resp *Response, err error)//Do发送http请求并且返回一个http响应,遵守client的策略,如重定向,cookies以及auth等.错误经常是由于策略引起的,当err是nil时,resp总会包含一个非nil的resp.body.当调用者读完resp.body之后应该关闭它,如果resp.body没有关闭,则Client底层RoundTripper将无法重用存在的TCP连接去服务接下来的请求,如果resp.body非nil,则必须对其进行关闭.通常来说,经常使用Get,Post,或者PostForm来替代Do.
func (c *Client) Get(url string) (resp *Response, err error)
//利用get方法请求指定的url.Get请求指定的页面信息，并返回实体主体。
func (c *Client) Head(url string) (resp *Response, err error)
//利用head方法请求指定的url，Head只返回页面的首部。
func (c *Client) Post(url string, bodyType string, body io.Reader) (resp *Response, err error)
//利用post方法请求指定的URl,如果body也是一个io.Closer,则在请求之后关闭它
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)
//利用post方法请求指定的url,利用data的key和value作为请求体.
```

http中Client客户端的参数设定解析如上面描述所示，Client具有Do，Get，Head，Post以及PostForm等方法。 其中Do方法可以对Request进行一系列的设定，而其他的对request设定较少。如果Client使用默认的Client，则其中的Get，Head，Post以及PostForm方法相当于默认的http.Get,http.Post,http.Head以及http.PostForm函数。举例说明其用法。其中Get，Head，Post以及PostForm用法如下：
```go
package main

import (
    "bytes"
    "fmt"
    "io/ioutil"
    "net/http"
    "net/url"
)

func main() {

    requestUrl := "http://www.baidu.com"
    // request, err := http.Get(requestUrl)
    // request, err := http.Head(requestUrl)
    postvalue := url.Values{
        "username": {"xiaoming"},
        "address":  {"beijing"},
        "subject":  {"Hello"},
        "from":     {"china"},
    }
    // request, err := http.PostForm(requestUrl, postvalue)

    body := bytes.NewBufferString(postvalue.Encode())
    request, err := http.Post(requestUrl, "text/html", body)  //Post方法
    if err != nil {
        fmt.Println(err)
    }

    defer request.Body.Close()
    fmt.Println(request.StatusCode)
    if request.StatusCode == 200 {
        rb, err := ioutil.ReadAll(request.Body)
        if err != nil {
            fmt.Println(rb)
        }
        fmt.Println(string(rb))
    }

}
```
Do方法可以灵活的对request进行配置，然后进行请求。利用http.Client以及http.NewRequest来模拟请求。模拟request中带有cookie的请求。  
```go
package main

import (
    // "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
)

func main() {
    client := &http.Client{}
    request, err := http.NewRequest("GET", "http://www.baidu.com", nil)
    if err != nil {
        fmt.Println(err)
    }
    cookie := &http.Cookie{Name: "userId", Value: strconv.Itoa(12345)}

    request.AddCookie(cookie) //request中添加cookie

    //设置request的header
    request.Header.Set("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
    request.Header.Set("Accept-Charset", "GBK,utf-8;q=0.7,*;q=0.3")
    request.Header.Set("Accept-Encoding", "gzip,deflate,sdch")
    request.Header.Set("Accept-Language", "zh-CN,zh;q=0.8")
    request.Header.Set("Cache-Control", "max-age=0")
    request.Header.Set("Connection", "keep-alive")

    response, err := client.Do(request)
    if err != nil {
        fmt.Println(err)
        return
    }

    defer response.Body.Close()
    fmt.Println(response.StatusCode)
    if response.StatusCode == 200 {
        r, err := ioutil.ReadAll(response.Body)
        if err != nil {
            fmt.Println(err)
        }
        fmt.Println(string(r))
    }
}
```

### **type CloseNotifier 接口**  
```go
type CloseNotifier//该接口被ResponseWriter用来实时检测底层连接是否已经断开.如果客户端已经断开连接,该机制可以在服务端响应之前取消二者之间的长连接.
type CloseNotifier interface {
        // 当客户端断开连接时,CloseNotifier接受一个通知
        CloseNotify() <-chan bool
}
```

CloseNotifier经常被ResponseWriter用来检测底层连接是否已经断开，举例如下：  

```go
func Test(w http.ResponseWriter, r *http.Request) {
    http.ServeFile(w, r, "test.go")
    var closenotify http.CloseNotifier
    closenotify = w.(http.CloseNotifier)
    select {
    case <-closenotify.CloseNotify():
        fmt.Println("cut")
    case <-time.After(time.Duration(100) * time.Second):
        fmt.Println("timeout")

    }
}
func main() {

    http.HandleFunc("/test", Test)
    err := http.ListenAndServe(":9999", nil)
    if err != nil {
        fmt.Println(err)
    }

}
```

### **type ConnState int**  
```go
type ConnState //表示客户端连接服务端的状态  
type ConnState int
```

其中ConnState常用状态变量如下:  
```go
const (
    // StateNew代表一个新的连接，将要立刻发送请求。
    // 连接从这个状态开始，然后转变为StateAlive或StateClosed。
    StateNew ConnState = iota
    // StateActive代表一个已经读取了请求数据1到多个字节的连接。
    // 用于StateAlive的Server.ConnState回调函数在将连接交付给处理器之前被触发，
    // 等到请求被处理完后，Server.ConnState回调函数再次被触发。
    // 在请求被处理后，连接状态改变为StateClosed、StateHijacked或StateIdle。
    StateActive
    // StateIdle代表一个已经处理完了请求、处在闲置状态、等待新请求的连接。
    // 连接状态可以从StateIdle改变为StateActive或StateClosed。
    StateIdle
    // 代表一个被劫持的连接。这是一个终止状态，不会转变为StateClosed。
    StateHijacked
    // StateClosed代表一个关闭的连接。
    // 这是一个终止状态。被劫持的连接不会转变为StateClosed。
    StateClosed
)
```

```go
func (c ConnState) String() string  //状态的字符形式表示
fmt.Println(http.StateActive)          //active
fmt.Println(http.StateActive.String()) //active
```

### **type Cookie struct**  

type Cookie//常用SetCooker用来给http的请求或者http的response设置cookie  

```go
type Cookie struct {
        Name       string  //名字
        Value      string  //值
        Path       string   //路径
        Domain     string   
        Expires    time.Time //过期时间
        RawExpires string

        // MaxAge=0 意味着 没有'Max-Age'属性指定.
        // MaxAge<0 意味着 立即删除cookie
        // MaxAge>0 意味着设定了MaxAge属性,并且其单位是秒
        MaxAge   int
        Secure   bool
        HttpOnly bool
        Raw      string
        Unparsed []string // 未解析的属性值对
}
```

func (c *Cookie) String() string//该函数返回cookie的序列化结果。如果只设置了Name和Value字段，序列化结果可用于HTTP请求的Cookie头或者HTTP回复的Set-Cookie头；如果设置了其他字段，序列化结果只能用于HTTP回复的Set-Cookie头。  

type CookieJar//在http请求中，CookieJar管理存储和使用cookies.Cookiejar的实现必须被多协程并发使用时是安全的.  

```go
type CookieJar interface {
        // SetCookies 处理从url接收到的cookie,是否存储这个cookies取决于jar的策略和实现
        SetCookies(u *url.URL, cookies []*Cookie)

        // Cookies 返回发送到指定url的cookies
        Cookies(u *url.URL) []*Cookie
}
```

### **type Dir string**  
type Dir//使用一个局限于指定目录树的本地文件系统实现一个文件系统.一个空目录被当做当前目录"."  

```go
type Dir string
func (d Dir) Open(name string) (File, error)
```

### **type File 接口**  

type File //File是通过FileSystem的Open方法返回的,并且能够被FileServer实现.该方法与*os.File行为表现一样.  

```go
type File interface {
        io.Closer
        io.Reader
        Readdir(count int) ([]os.FileInfo, error)
        Seek(offset int64, whence int) (int64, error)
        Stat() (os.FileInfo, error)
}
```

**type FileSystem 接口**  

type FileSystem//实现了对一系列指定文件的访问,其中文件路径之间通过分隔符进行分割.  

```go
type FileSystem interface {
        Open(name string) (File, error)
}
```

### **type Flusher 接口**  

type Flusher //responsewriters允许http控制器将缓存数据刷新入client.然而如果client是通过http代理连接服务器,这个缓存数据也可能是在整个response结束后才能到达客户端.  

```go
type Flusher interface {
        // Flush将任何缓存数据发送到client
        Flush()
}
```

### **type Handler 接口**  

type Handler //实现Handler接口的对象可以注册到HTTP服务端，为指定的路径或者子树提供服务。ServeHTTP应该将回复的header和数据写入ResponseWriter接口然后返回。返回意味着该请求已经结束，HTTP服务端可以转移向该连接上的下一个请求。  

//如果ServeHTTP崩溃panic,那么ServeHTTP的调用者假定这个panic的影响与活动请求是隔离的,二者互不影响.调用者恢复panic,将stack trace记录到错误日志中,然后挂起这个连接.

```go
type Handler interface {
        ServeHTTP(ResponseWriter, *Request)
}
```
func FileServer(root FileSystem) Handler // FileServer返回一个使用FileSystem接口提供文件访问服务的HTTP处理器。可以使用httpDir来使用操作系统的FileSystem接口实现。其主要用来实现静态文件的展示。  

func NotFoundHandler() Handler //返回一个简单的请求处理器,该处理器对任何请求都会返回"404 page not found"  

func RedirectHandler(url string, code int) Handler//使用给定的状态码将它接受到的任何请求都重定向到给定的url  

func StripPrefix(prefix string, h Handler) Handler//将请求url.path中移出指定的前缀,然后将省下的请求交给handler h来处理,对于那些不是以指定前缀开始的路径请求,该函数返回一个http 404 not found 的错误.  

func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler //具有超时限制的handler,该函数返回的新Handler调用h中的ServerHTTP来处理每次请求,但是如果一次调用超出时间限制,那么就会返回给请求者一个503服务请求不可达的消息,并且在ResponseWriter返回超时错误.  

其中FileServer经常和StripPrefix一起连用，用来实现静态文件展示，举例如下：  

```go
func main() {
    http.Handle("/test/", http.FileServer(http.Dir("/home/work/"))) ///home/work/test/中必须有内容
    http.Handle("/download/", http.StripPrefix("/download/", http.FileServer(http.Dir("/home/work/"))))
    http.Handle("/tmpfiles/", http.StripPrefix("/tmpfiles/", http.FileServer(http.Dir("/tmp")))) //127.0.0.1:9999/tmpfiles/访问的本地文件/tmp中的内容
    http.ListenAndServe(":9999", nil)
}
```

### **type HandlerFunc**  

type HandlerFunc//HandlerFunc type是一个适配器，通过类型转换我们可以将普通的函数作为HTTP处理器使用。如果f是一个具有适当签名的函数，HandlerFunc(f)通过调用f实现了Handler接口。  

```go
type HandlerFunc func(ResponseWriter, *Request)
```

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) //ServeHttp调用f(w,r)  

type Header//代表在http header中的key-value对  

```go
type Header map[string][]string
```
```go
func (h Header) Add(key, value string)//将key,value组成键值对添加到header中
func (h Header) Del(key string)  //header中删除对应的key-value对
func (h Header) Get(key string) string //获取指定key的第一个value,如果key没有对应的value,则返回"",为了获取一个key的多个值,用CanonicalHeaderKey来获取标准规范格式.
func (h Header) Set(key, value string) //给一个key设定为相应的value.
func (h Header) Write(w io.Writer) error//利用线格式来写header
func (h Header) WriteSubset(w io.Writer, exclude map[string]bool) error//利用线模式写header,如果exclude不为nil,则在exclude中包含的exclude[key] == true不被写入.
```

### **type Hijacker 接口**  

```go
type Hijacker interface {
        // Hijack让调用者接管连接,在调用Hijack()后,http server库将不再对该连接进行处理,对于该连接的管理和关闭责任将由调用者接管.
        Hijack() (net.Conn, *bufio.ReadWriter, error) //conn表示连接对象，bufrw代表该连接的读写缓存对象。
}
```
eg.  
```go
func HiJack(w http.ResponseWriter, r *http.Request) {
    hj, ok := w.(http.Hijacker)
    if !ok {
        http.Error(w, "webserver doesn't support hijacking", http.StatusInternalServerError)
        return
    }
    conn, bufrw, err := hj.Hijack()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    defer conn.Close()
    bufrw.WriteString("Now we're speaking raw TCP. Say hi: \n")
    bufrw.Flush()

    fmt.Fprintf(bufrw, "You said: %s Bye.\n", "Good")
    bufrw.Flush()
}
func main() {
    http.HandleFunc("/hijack", HiJack)
    err := http.ListenAndServe(":9999", nil)
    if err != nil {
        fmt.Println(err)
    }
}
```

### **type ProtocolError**  

type ProtocolError //http请求解析错误  

```go
func (err *ProtocolError) Error() string
```

### **type Reques struct**  

type Request//代表客户端给服务器端发送的一个请求.该字段在服务器端和客户端使用时区别很大.  

```go
type Request struct {
    // Method指定HTTP方法(GET,POST,PUT等)默认使用GET方法。
    Method string
    // URL在服务端表示被请求的URI(uniform resource identifier,统一资源标识符)，在客户端表示要访问的URL。
    // 在服务端,URL字段是解析请求行的URI（保存在RequestURI字段）得到的,对大多数请求来说,除了Path和RawQuery之外的字段都是空字符串。
    // 在客户端,URL的Host字段指定了要连接的服务器,而Request的Host字段指定要发送的HTTP请求的Host头的值。
    URL *url.URL
    // 接收到的请求的协议版本。client的Request总是使用HTTP/1.1
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0
    // Header字段用来表示HTTP请求的头域。如果header（多行键值对格式）为：
    //    accept-encoding: gzip, deflate
    //    Accept-Language: en-us
    //    Connection: keep-alive
    // 则：
    //    Header = map[string][]string{
    //        "Accept-Encoding": {"gzip, deflate"},
    //        "Accept-Language": {"en-us"},
    //        "Connection": {"keep-alive"},
    //    }
    // HTTP规定header的键名（头名）是区分大小写的，请求的解析器通过规范化头域的键名来实现这点,即首字母大写,其他字母小写,通过"-"进行分割。
    // 在客户端的请求，可能会被自动添加或重写Header中的特定的头，参见Request.Write方法。
    Header Header
    // Body是请求的主体.对于客户端请求来说,一个nil body意味着没有body,http Client的Transport字段负责调用Body的Close方法。
    // 在服务端，Body字段总是非nil的；但在没有主体时，读取Body会立刻返回EOF.Server会关闭请求主体，而ServeHTTP处理器不需要关闭Body字段。
    Body io.ReadCloser
    // ContentLength记录相关内容的长度.如果为-1,表示长度未知,如果values>=0，表示可以从Body字段读取ContentLength字节数据。
    // 在客户端,如果Body非nil而该字段为0,表示不知道Body的长度。
    ContentLength int64
    // TransferEncoding按从最外到最里的顺序列出传输编码，空切片表示"identity"编码。
    // 本字段一般会被忽略。当发送或接受请求时，会自动添加或移除"chunked"传输编码。
    TransferEncoding []string
    // Close在服务端指定是否在回复请求后关闭连接，在客户端指定是否在发送请求后关闭连接。
    Close bool
    // 对于服务器端请求,Host指定URL指向的主机,可能的格式是host:port.对于客户请求,Host用来重写请求的Host头,如过该字段为""，Request.Write方法会使用URL.Host来进行赋值。
    Host string
    // Form是解析好的表单数据，包括URL字段的query参数和POST或PUT的表单数据.本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    Form url.Values
    // PostForm是解析好的POST或PUT的表单数据.本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    PostForm url.Values
    // MultipartForm是解析好的多部件表单，包括上传的文件.本字段只有在调用ParseMultipartForm后才有效。http客户端中会忽略MultipartForm并且使用Body替代
    MultipartForm *multipart.Form
    // Trailer指定了在发送请求体之后额外的headers,在服务端，Trailer字段必须初始化为只有trailer键，所有键都对应nil值。
    // （客户端会声明哪些trailer会发送）在处理器从Body读取时，不能使用本字段.在从Body的读取返回EOF后，Trailer字段会被更新完毕并包含非nil的值。
    // （如果客户端发送了这些键值对），此时才可以访问本字段。
    // 在客户端，Trail必须初始化为一个包含将要发送的键值对的映射.(值可以是nil或其终值),ContentLength字段必须是0或-1，以启用"chunked"传输编码发送请求。
    // 在开始发送请求后,Trailer可以在读取请求主体期间被修改，一旦请求主体返回EOF，调用者就不可再修改Trailer。
    // 几乎没有HTTP客户端、服务端或代理支持HTTP trailer。
    Trailer Header
    // RemoteAddr允许HTTP服务器和其他软件记录该请求的来源地址,该字段经常用于日志.本字段不是ReadRequest函数填写的，也没有定义格式。
    // 本包的HTTP服务器会在调用处理器之前设置RemoteAddr为"IP:port"格式的地址.客户端会忽略请求中的RemoteAddr字段。
    RemoteAddr string
    // RequestURI是客户端发送到服务端的请求中未修改的URI(参见RFC 2616,Section 5.1),如果在http请求中设置该字段便会报错.
    RequestURI string
    // TLS字段允许HTTP服务器和其他软件记录接收到该请求的TLS连接的信息,本字段不是ReadRequest函数填写的。
    // 对启用了TLS的连接，本包的HTTP服务器会在调用处理器之前设置TLS字段，否则将设TLS为nil。
    // 客户端会忽略请求中的TLS字段。
    TLS *tls.ConnectionState
}
```
func NewRequest(method, urlStr string, body io.Reader) (*Request, error) //利用指定的method,url以及可选的body返回一个新的请求.如果body参数实现了io.Closer接口，Request返回值的Body 字段会被设置为body，并会被Client类型的Do、Post和PostForm方法以及Transport.RoundTrip方法关闭。  

func ReadRequest(b *bufio.Reader) (req *Request, err error)//从b中读取和解析一个请求.  

func (r *Request) AddCookie(c *Cookie) //给request添加cookie,AddCookie向请求中添加一个cookie.按照RFC 6265 section 5.4的规则,AddCookie不会添加超过一个Cookie头字段.这表示所有的cookie都写在同一行,用分号分隔（cookie内部用逗号分隔属性）  

func (r *Request) Cookie(name string) (*Cookie, error)//返回request中指定名name的cookie，如果没有发现，返回ErrNoCookie  

func (r *Request) Cookies() []*Cookie//返回该请求的所有cookies  

func (r *Request) SetBasicAuth(username, password string)//利用提供的用户名和密码给http基本权限提供具有一定权限的header。当使用http基本授权时，用户名和密码是不加密的  

func (r *Request) UserAgent() string//如果在request中发送，该函数返回客户端的user-Agent

用法eg.  

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func Test() {
    req, err := http.NewRequest("GET", "http://www.baidu.com", nil)
    if err != nil {
        fmt.Println(err)
        return
    }

    req.SetBasicAuth("test", "123456")
    fmt.Println(req.Proto)
    cookie := &http.Cookie{
        Name:  "test",
        Value: "12",
    }
    req.AddCookie(cookie)                     //添加cookie
    fmt.Println(req.Cookie("test"))           //获取cookie
    fmt.Println(req.Cookies())                //获取cookies
    req.Header.Set("User-Agent", "useragent") //设定ua
    fmt.Println(req.UserAgent())

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println(err)
        return
    }
    defer resp.Body.Close()
    if resp.StatusCode == 200 {
        content, _ := ioutil.ReadAll(resp.Body)
        fmt.Println(string(content))
    }

}
func main() {
    Test()
}
```

func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)//对于指定格式的key，FormFile返回符合条件的第一个文件，如果有必要的话，该函数会调用ParseMultipartForm和ParseForm。  

func (r *Request) FormValue(key string) string//返回key获取的队列中第一个值。在查询过程中post和put中的主题参数优先级高于url中的value。为了访问相同key的多个值，调用ParseForm然后直接检查RequestForm。  

func (r *Request) MultipartReader() (*multipart.Reader, error)//如果这是一个有多部分组成的post请求，该函数将会返回一个MIME 多部分reader，否则的话将会返回一个nil和error。使用本函数代替ParseMultipartForm可以将请求body当做流stream来处理。  

func (r *Request) ParseForm() error//解析URL中的查询字符串，并将解析结果更新到r.Form字段。对于POST或PUT请求，ParseForm还会将body当作表单解析，并将结果既更新到r.PostForm也更新到r.Form。解析结果中，POST或PUT请求主体要优先于URL查询字符串（同名变量，主体的值在查询字符串的值前面）。如果请求的主体的大小没有被MaxBytesReader函数设定限制，其大小默认限制为开头10MB。ParseMultipartForm会自动调用ParseForm。重复调用本方法是无意义的。  

func (r *Request) ParseMultipartForm(maxMemory int64) error //ParseMultipartForm将请求的主体作为multipart/form-data解析。请求的整个主体都会被解析，得到的文件记录最多 maxMemery字节保存在内存，其余部分保存在硬盘的temp文件里。如果必要，ParseMultipartForm会自行调用 ParseForm。重复调用本方法是无意义的。  

func (r *Request) PostFormValue(key string) string//返回post或者put请求body指定元素的第一个值，其中url中的参数被忽略。  

func (r *Request) ProtoAtLeast(major, minor int) bool//检测在request中使用的http协议是否至少是major.minor  

func (r *Request) Referer() string//如果request中有refer，那么refer返回相应的url。Referer在request中是拼错的，这个错误从http初期就已经存在了。该值也可以从Headermap中利用Header["Referer"]获取；在使用过程中利用Referer这个方法而不是map的形式的好处是在编译过程中可以检查方法的错误，而无法检查map中key的错误。  

 func (r *Request) Write(w io.Writer) error//Write方法以有线格式将HTTP/1.1请求写入w（用于将请求写入下层TCPConn等）。本方法会考虑请求的如下字段：Host URL Method (defaults to "GET") Header ContentLength TransferEncoding Body
　如果存在Body，ContentLength字段<= 0且TransferEncoding字段未显式设置为["identity"]，Write方法会显式添加"Transfer-Encoding: chunked"到请求的头域。Body字段会在发送完请求后关闭。  

 func (r *Request) WriteProxy(w io.Writer) error//该函数与Write方法类似，但是该方法写的request是按照http代理的格式去写。尤其是，按照RFC 2616 Section 5.1.2，WriteProxy会使用绝对URI（包括协议和主机名）来初始化请求的第1行（Request-URI行）。无论何种情况，WriteProxy都会使用r.Host或r.URL.Host设置Host头。


### **type Response struct**  

type Response//指对于一个http请求的响应response  

```go
type Response struct {
    Status     string // 例如"200 OK"
    StatusCode int    // 例如200
    Proto      string // 例如"HTTP/1.0"
    ProtoMajor int    // 主协议号：例如1
    ProtoMinor int    // 副协议号：例如0
    // Header保管header的key values，如果response中有多个header中具有相同的key，那么Header中保存为该键对应用逗号分隔串联起来的这些头的值// 被本结构体中的其他字段复制保管的头（如ContentLength）会从Header中删掉。Header中的键都是规范化的，参见CanonicalHeaderKey函数
    Header Header
    // Body代表response的主体。http的client和Transport确保这个body永远非nil，即使response没有body或body长度为0。调用者也需要关闭这个body// 如果服务端采用"chunked"传输编码发送的回复，Body字段会自动进行解码。
    Body io.ReadCloser
    // ContentLength记录相关内容的长度。
    // 其值为-1表示长度未知（采用chunked传输编码）
    // 除非对应的Request.Method是"HEAD"，其值>=0表示可以从Body读取的字节数
    ContentLength int64
    // TransferEncoding按从最外到最里的顺序列出传输编码，空切片表示"identity"编码。
    TransferEncoding []string
    // Close记录头域是否指定应在读取完主体后关闭连接。（即Connection头）
    // 该值是给客户端的建议，Response.Write方法的ReadResponse函数都不会关闭连接。
    Close bool
    // Trailer字段保存和头域相同格式的trailer键值对，和Header字段相同类型
    Trailer Header
    // Request是用来获取此回复的请求，Request的Body字段是nil（因为已经被用掉了）这个字段是被Client类型发出请求并获得回复后填充的
    Request *Request
    // TLS包含接收到该回复的TLS连接的信息。 对未加密的回复，本字段为nil。返回的指针是被（同一TLS连接接收到的）回复共享的，不应被修改。
    TLS *tls.ConnectionState
}
```

func Get(url string) (resp *Response, err error)//利用GET方法对一个指定的URL进行请求，如果response是如下重定向中的一个代码，则Get之后将会调用重定向内容，最多10次重定向。  

```go
301 (永久重定向，告诉客户端以后应该从新地址访问)
302 (暂时性重定向，作为HTTP1.0的标准，以前叫做Moved Temporarily，现在叫做Found。现在使用只是为了兼容性处理，包括PHP的默认Location重定向用到也是302)，注：303和307其实是对302的细化。
303 (对于Post请求，它表示请求已经被处理，客户端可以接着使用GET方法去请求Location里的URl)
307 (临时重定向，对于Post请求，表示请求还没有被处理，客户端应该向Location里的URL重新发起Post请求)
```

//如果有太多次重定向或者有一个http协议错误将会导致错误。当err为nil时，resp总是包含一个非nil的resp.body，Get是对DefaultClient.Get的一个包装。  

func Head(url string) (resp *Response, err error)//该函数功能见net中Head方法功能。该方法与默认的defaultClient中Head方法一致。  

func Post(url string, bodyType string, body io.Reader) (resp *Response, err error)//该方法与默认的defaultClient中Post方法一致。  

func PostForm(url string, data url.Values) (resp *Response, err error)//该方法与默认的defaultClient中PostForm方法一致。

func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)//ReadResponse从r读取并返回一个HTTP 回复。req参数是可选的，指定该回复对应的请求（即是对该请求的回复）。如果是nil，将假设请 求是GET请求。客户端必须在结束resp.Body的读取后关闭它。读取完毕并关闭后，客户端可以检查resp.Trailer字段获取回复的 trailer的键值对。（本函数主要用在客户端从下层获取回复）  

func (r *Response) Cookies() []*Cookie//解析cookie并返回在header中利用set-Cookie设定的cookie值。  

func (r *Response) Location() (*url.URL, error)//返回response中Location的header值的url。如果该值存在的话，则对于请求问题可以解决相对重定向的问题，如果该值为nil，则返回ErrNOLocation的错误。  

func (r *Response) ProtoAtLeast(major, minor int) bool//判定在response中使用的http协议是否至少是major.minor的形式。  

func (r *Response) Write(w io.Writer) error//将response中信息按照线性格式写入w中。

**type ResponseWriter 接口**  

type ResponseWriter//该接口被http handler用来构建一个http response  

```go
type ResponseWriter interface {
    // Header返回一个Header类型值，该值会被WriteHeader方法发送.在调用WriteHeader或Write方法后再改变header值是不起作用的。
    Header() Header
    // WriteHeader该方法发送HTTP回复的头域和状态码。如果没有被显式调用，第一次调用Write时会触发隐式调用WriteHeader(http.StatusOK)
    // 因此，显示调用WriterHeader主要用于发送错误状态码。
    WriteHeader(int)
    // Write向连接中写入数据，该数据作为HTTP response的一部分。如果被调用时还没有调用WriteHeader，本方法会先调用WriteHeader(http.StatusOK)
    // 如果Header中没有"Content-Type"键，本方法会使用包函数DetectContentType检查数据的前512字节，将返回值作为该键的值。
    Write([]byte) (int, error)
}
```

### **type RoundTripper 接口**  

type RoundTripper//该函数是一个执行简单http事务的接口，该接口在被多协程并发使用时必须是安全的。  

```go
type RoundTripper interface {
    // RoundTrip执行单次HTTP事务，返回request的response，RoundTrip不应试图解析该回复。
    // 尤其要注意，只要RoundTrip获得了一个回复，不管该回复的HTTP状态码如何，它必须将返回值err设置为nil。
    // 非nil的返回值err应该留给获取回复失败的情况。类似的，RoundTrip不能试图管理高层协议，如重定向、认证或者cookie。
    // RoundTrip除了从请求的主体读取并关闭主体之外，不能够对请求做任何修改，包括（请求的）错误。
    // RoundTrip函数接收的请求的URL和Header字段必须保证是初始化了的。
    RoundTrip(*Request) (*Response, error)
}
```

func NewFileTransport(fs FileSystem) RoundTripper //该函数返回一个RoundTripper接口，服务指定的文件系统。 返回的RoundTripper接口会忽略接收的请求中的URL主机及其他绝大多数属性。该函数的典型应用是给Transport类型的值注册"file"协议。如下所示：  

```go
t := &http.Transport{}
t.RegisterProtocol("file", http.NewFileTransport(http.Dir("/")))
c := &http.Client{Transport: t}
res, err := c.Get("file:///etc/passwd")
...
```

### **type ServeMux **   

type ServeMux //该函数是一个http请求多路复用器，它将每一个请求的URL和一个注册模式的列表进行匹配，然后调用和URL最匹配的模式的处理器进行后续操作。模式是固定的、由根开始的路径，如"/favicon.ico"，或由根开始的子树，如"/images/" （注意结尾的斜杠）。较长的模式优先于较短的模式，因此如果模式"/images/"和"/images/thumbnails/"都注册了处理器，后一 个处理器会用于路径以"/images/thumbnails/"开始的请求，前一个处理器会接收到其余的路径在"/images/"子树下的请求。
注意，因为以斜杠结尾的模式代表一个由根开始的子树，模式"/"会匹配所有的未被其他注册的模式匹配的路径，而不仅仅是路径"/"。  

模式也能（可选地）以主机名开始，表示只匹配该主机上的路径。指定主机的模式优先于一般的模式，因此一个注册了两个模式"/codesearch"和"codesearch.google.com/"的处理器不会接管目标为"http://www.google.com/"的请求。  

ServeMux还会负责对URL路径的过滤，将任何路径中包含"."或".."元素的请求重定向到等价的没有这两种元素的URL。（参见path.Clean函数）  

func NewServeMux() *ServeMux //初始化一个新的ServeMux  

func (mux *ServeMux) Handle(pattern string, handler Handler) //将handler注册为指定的模式，如果该模式已经有了handler，则会出错panic。  

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))//将handler注册为指定的模式

func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) //根据指定的r.Method,r.Host以及r.RUL.Path返回一个用来处理给定请求的handler。该函数总是返回一个非nil的 handler，如果path不是一个规范格式，则handler会重定向到其规范path。Handler总是返回匹配该请求的的已注册模式；在内建重 定向处理器的情况下，pattern会在重定向后进行匹配。如果没有已注册模式可以应用于该请求，本方法将返回一个内建的"404 page not found"处理器和一个空字符串模式。

func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)  //该函数用于将最接近请求url模式的handler分配给指定的请求。  

举例说明servemux的用法：
```go
package main

import (
    "fmt"
    "net/http"
)

func Test(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "just for test!")
}
func main() {
    mux := http.NewServeMux()
    mux.Handle("/", http.FileServer(http.Dir("/home")))
    mux.HandleFunc("/test", Test)
    err := http.ListenAndServe(":9999", mux)
    if err != nil {
        fmt.Println(err)
    }
}
```

### **type Server struct**  

type Server //该结构体定义一些用来运行HTTP Server的参数，如果Server默认为0的话，那么这也是一个有效的配置。  

```go
type Server struct {
    Addr           string        // 监听的TCP地址，如果为空字符串会使用":http"
    Handler        Handler       // 调用的处理器，如为nil会调用http.DefaultServeMux
    ReadTimeout    time.Duration // 请求的读取操作在超时前的最大持续时间
    WriteTimeout   time.Duration // 回复的写入操作在超时前的最大持续时间
    MaxHeaderBytes int           // 请求的头域最大长度，如为0则用DefaultMaxHeaderBytes
    TLSConfig      *tls.Config   // 可选的TLS配置，用于ListenAndServeTLS方法
    // TLSNextProto（可选地）指定一个函数来在一个NPN型协议升级出现时接管TLS连接的所有权。
    // 映射的键为商谈的协议名；映射的值为函数，该函数的Handler参数应处理HTTP请求，
    // 并且初始化Handler.ServeHTTP的*Request参数的TLS和RemoteAddr字段（如果未设置）。
    // 连接在函数返回时会自动关闭。
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
    // ConnState字段指定一个可选的回调函数，该函数会在一个与客户端的连接改变状态时被调用。
    // 参见ConnState类型和相关常数获取细节。
    ConnState func(net.Conn, ConnState)
    // ErrorLog指定一个可选的日志记录器，用于记录接收连接时的错误和处理器不正常的行为。
    // 如果本字段为nil，日志会通过log包的标准日志记录器写入os.Stderr。
    ErrorLog *log.Logger
    // 内含隐藏或非导出字段
}
```

func (srv *Server) ListenAndServe() error//监听TCP网络地址srv.Addr然后调用Serve来处理接下来连接的请求。如果srv.Addr是空的话，则使用“:http”。  

func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error//ListenAndServeTLS监听srv.Addr确定的TCP地址，并且会调用Serve方法处理接收到的连接。必须提供证书文件和对应的私钥文 件。如果证书是由权威机构签发的，certFile参数必须是顺序串联的服务端证书和CA证书。如果srv.Addr为空字符串，会使 用":https"。  

func (srv *Server) Serve(l net.Listener) error//接受Listener l的连接，创建一个新的服务协程。该服务协程读取请求然后调用srv.Handler来应答。实际上就是实现了对某个端口进行监听，然后创建相应的连接。  

func (s *Server) SetKeepAlivesEnabled(v bool)//该函数控制是否http的keep-alives能够使用，默认情况下，keep-alives总是可用的。只有资源非常紧张的环境或者服务端在关闭进程中时，才应该关闭该功能。  

举例说明Server的用法：  

```go
package main

import (
    "fmt"
    "net/http"
)

func Test(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "just for test!")
}
func main() {
    newserver := http.Server{
        Addr:         ":9992",
        ReadTimeout:  0,
        WriteTimeout: 0,
    }

    mux := http.NewServeMux()
    mux.Handle("/", http.FileServer(http.Dir("/home")))
    mux.HandleFunc("/test", Test)

    newserver.Handler = mux
    err := newserver.ListenAndServe()
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(err)
    // err := http.ListenAndServe(":9999", mux)
    // if err != nil {
    //     fmt.Println(err)
    // }
}
```

### **type Transport struct**  

type Transport  //该结构体实现了RoundTripper接口，支持HTTP，HTTPS以及HTTP代理，TranSport也能缓存连接供将来使用。  

```go
type Transport struct {
    // Proxy指定一个对给定请求返回代理的函数。如果该函数返回了非nil的错误值，请求的执行就会中断并返回该错误。
    // 如果Proxy为nil或返回nil的*URL值，将不使用代理。
    Proxy func(*Request) (*url.URL, error)
    // Dial指定创建未加密TCP连接的dial函数。如果Dial为nil，会使用net.Dial。
    Dial func(network, addr string) (net.Conn, error)
　　// DialTls利用一个可选的dial函数来为非代理的https请求创建一个TLS连接。如果DialTLS为nil的话，那么使用Dial和TLSClientConfig。
　　//如果DialTLS被设定，那么Dial钩子不被用于HTTPS请求和TLSClientConfig并且TLSHandshakeTimeout被忽略。返回的net.conn默认已经经过了TLS握手协议。
　　DialTLS func(network, addr string) (net.Conn, error) 
    // TLSClientConfig指定用于tls.Client的TLS配置信息。如果该字段为nil，会使用默认的配置信息。
    TLSClientConfig *tls.Config
    // TLSHandshakeTimeout指定等待TLS握手完成的最长时间。零值表示不设置超时。
    TLSHandshakeTimeout time.Duration
    // 如果DisableKeepAlives为真，不同HTTP请求之间TCP连接的重用将被阻止。
    DisableKeepAlives bool
    // 如果DisableCompression为真，会禁止Transport在请求中没有Accept-Encoding头时，
    // 主动添加"Accept-Encoding: gzip"头，以获取压缩数据。
    // 如果Transport自己请求gzip并得到了压缩后的回复，它会主动解压缩回复的主体。
    // 但如果用户显式的请求gzip压缩数据，Transport是不会主动解压缩的。
    DisableCompression bool
    // 如果MaxIdleConnsPerHost不为0，会控制每个主机下的最大闲置连接数目。
    // 如果MaxIdleConnsPerHost为0，会使用DefaultMaxIdleConnsPerHost。
    MaxIdleConnsPerHost int
    // ResponseHeaderTimeout指定在发送完请求（包括其可能的主体）之后，
    // 等待接收服务端的回复的头域的最大时间。零值表示不设置超时。
    // 该时间不包括读取回复主体的时间。
    ResponseHeaderTimeout time.Duration
}
```

func (t *Transport) CancelRequest(req *Request)//通过关闭连接来取消传送中的请求。  

func (t *Transport) CloseIdleConnections()//关闭所有之前请求但目前处于空闲状态的连接。该方法并不中断任何正在使用的连接。  

func (t *Transport) RegisterProtocol(scheme string, rt RoundTripper)//RegisterProtocol注册一个新的名为scheme的协议。t会将使用scheme协议的请求转交给rt。rt有责任模拟HTTP请求的语义。RegisterProtocol可以被其他包用于提供"ftp"或"file"等协议的实现。  

func (t *Transport) RoundTrip(req *Request) (resp *Response, err error)//该函数实现了RoundTripper接口，对于高层http客户端支持，例如处理cookies以及重定向，查看Get，Post以及Client类型。


---

# `io/ioutil` 包  

## 函数  

### Discard  
Discard 是一个 io.Writer 接口，调用它的 Write 方法将不做任何事情并且始终成功返回。  
```go
var Discard io.Writer = devNull(0)
```

### ReadAll  

ReadAll 读取 r 中的所有数据，返回读取的数据和遇到的错误。  
如果读取成功，则 err 返回 nil，而不是 EOF，因为 ReadAll 定义为读取所有数据，所以不会把 EOF 当做错误处理。  

```go
func ReadAll(r io.Reader) ([]byte, error)
```

### ReadFile  

ReadFile 读取文件中的所有数据，返回读取的数据和遇到的错误。  

如果读取成功，则 err 返回 nil，而不是 EOF。

```go
func ReadFile(filename string) ([]byte, error)
```

### WriteFile  

WriteFile 向文件中写入数据，写入前会清空文件。  

如果文件不存在，则会以指定的权限创建该文件。  

返回遇到的错误。  

```go
func WriteFile(filename string, data []byte, perm os.FileMode) error
```

### ReadDir  

ReadDir 读取指定目录中的所有目录和文件（不包括子目录）。  

返回读取到的文件信息列表和遇到的错误，列表是经过排序的。  

```go
func ReadDir(dirname string) ([]os.FileInfo, error)
```

### NopCloser  

NopCloser 将 r 包装为一个 ReadCloser 类型，但 Close 方法不做任何事情。  

```go
func NopCloser(r io.Reader) io.ReadCloser
```

### TempFile  

TempFile 在 dir 目录中创建一个以 prefix 为前缀的临时文件，并将其以读写模式打开。返回创建的文件对象和遇到的错误。  

如果 dir 为空，则在默认的临时目录中创建文件（参见 os.TempDir），多次调用会创建不同的临时文件，调用者可以通过 f.Name() 获取文件的完整路径。  

调用本函数所创建的临时文件，应该由调用者自己删除。  

```go
func TempFile(dir, prefix string) (f *os.File, err error)
```

### TempDir  
TempDir 功能同 TempFile，只不过创建的是目录，返回目录的完整路径。  

```go
func TempDir(dir, prefix string) (name string, err error)
```

### 例子 

读取目录：  

```go
func main() {
    rd, err := ioutil.ReadDir("/")
    fmt.Println(err)
    for _, fi := range rd {
        if fi.IsDir() {
            fmt.Printf("[%s]\n", fi.Name())

        } else {
            fmt.Println(fi.Name())
        }
    }
}
```

临时目录、临时文件：  

```go
func main() {
    // 创建临时目录
    dir, err := ioutil.TempDir("", "Test")
    if err != nil {
        fmt.Println(err)
    }
    defer os.Remove(dir) // 用完删除
    fmt.Printf("%s\n", dir)

    // 创建临时文件
    f, err := ioutil.TempFile(dir, "Test")
    if err != nil {
        fmt.Println(err)
    }
    defer os.Remove(f.Name()) // 用完删除
    fmt.Printf("%s\n", f.Name())
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

# `runtime` 包  
runtime 包 提供了运行时与系统的交互,比如控制协程函数，触发垃圾立即回收等等底层操作  

## 函数  
### 获取GO的信息
```go
func GOROOT() string    // 获取GOROOT环境变量
func Version() string   // 获取GO的版本号
func NumCPU() int       // 获取本机逻辑CPU个数
func GOMAXPROCS(n int) int  // 设置最大可同时执行的最大CPU数
// 设置可同时执行的最大CPU数，并返回先前的设置。 
// 若 n < 1，它就不会更改当前设置。
```

### 设置cpu profile记录的速率  
```go
func SetCPUProfileRate(hz int)
```
SetCPUProfileRate设置CPU profile记录的速率为平均每秒hz次。如果hz<=0，SetCPUProfileRate会关闭profile的记录。如果记录器在执行，该速率必须在关闭之后才能修改。  

绝大多数使用者应使用runtime/pprof包或testing包的-test.cpuprofile选项而非直接使用SetCPUProfileRate  

### 立即执行一次垃圾回收  
```go
func GC()
```

### 给变量绑定方法，当垃圾回收的时候进行监听  
```go
func SetFinalizer(x, f interface{})
```
注意x必须是指针类型,f 函数的参数一定要和x保持一致,或者写interface{},不然程序会报错  
```go
type Student struct {
  name string
}
func main() {
  var i *Student = new(Student)
  runtime.SetFinalizer(i, func(i *Student) {
   println("垃圾回收了")
  })
  runtime.GC()
  time.Sleep(time.Second)
}
```

### 查看内存申请和分配统计信息  
```go
func ReadMemStats(m *MemStats)

type MemStats struct {
    // 一般统计
    Alloc      uint64 // 已申请且仍在使用的字节数
    TotalAlloc uint64 // 已申请的总字节数（已释放的部分也算在内）
    Sys        uint64 // 从系统中获取的字节数（下面XxxSys之和）
    Lookups    uint64 // 指针查找的次数
    Mallocs    uint64 // 申请内存的次数
    Frees      uint64 // 释放内存的次数
    // 主分配堆统计
    HeapAlloc    uint64 // 已申请且仍在使用的字节数
    HeapSys      uint64 // 从系统中获取的字节数
    HeapIdle     uint64 // 闲置span中的字节数
    HeapInuse    uint64 // 非闲置span中的字节数
    HeapReleased uint64 // 释放到系统的字节数
    HeapObjects  uint64 // 已分配对象的总个数
    // L低层次、大小固定的结构体分配器统计，Inuse为正在使用的字节数，Sys为从系统获取的字节数
    StackInuse  uint64 // 引导程序的堆栈
    StackSys    uint64
    MSpanInuse  uint64 // mspan结构体
    MSpanSys    uint64
    MCacheInuse uint64 // mcache结构体
    MCacheSys   uint64
    BuckHashSys uint64 // profile桶散列表
    GCSys       uint64 // GC元数据
    OtherSys    uint64 // 其他系统申请
    // 垃圾收集器统计
    NextGC       uint64 // 会在HeapAlloc字段到达该值（字节数）时运行下次GC
    LastGC       uint64 // 上次运行的绝对时间（纳秒）
    PauseTotalNs uint64
    PauseNs      [256]uint64 // 近期GC暂停时间的循环缓冲，最近一次在[(NumGC+255)%256]
    NumGC        uint32
    EnableGC     bool
    DebugGC      bool
    // 每次申请的字节数的统计，61是C代码中的尺寸分级数
    BySize [61]struct {
        Size    uint32
        Mallocs uint64
        Frees   uint64
    }
}
```

### 查看程序  
```go
func (r *MemProfileRecord) InUseBytes() int64   // InUseBytes返回正在使用的字节数（AllocBytes – FreeBytes）
func (r *MemProfileRecord) InUseObjects() int64 //InUseObjects返回正在使用的对象数（AllocObjects - FreeObjects）
func (r *MemProfileRecord) Stack() []uintptr    // Stack返回关联至此记录的调用栈踪迹，即r.Stack0的前缀。
func MemProfile(p []MemProfileRecord, inuseZero bool) (n int, ok bool)
// 获取内存profile记录历史
```
MemProfile返回当前内存profile中的记录数n。若len(p)>=n，MemProfile会将此分析报告复制到p中并返回(n, true)；如果len(p)< n，MemProfile则不会更改p，而只返回(n, false)。  

如果inuseZero为真，该profile就会包含无效分配记录（其中r.AllocBytes>0，而r.AllocBytes==r.FreeBytes。这些内存都是被申请后又释放回运行时环境的）。  

大多数调用者应当使用runtime/pprof包或testing包的-test.memprofile标记，而非直接调用MemProfile  

### 执行一个断点  
```go
func Breakpoint()
runtime.Breakpoint()
```

###  获取程序调用go协程的栈踪迹历史  
```go
func Stack(buf []byte, all bool) int
```
Stack将调用其的go程的调用栈踪迹格式化后写入到buf中并返回写入的字节数。若all为true，函数会在写入当前go程的踪迹信息后，将其它所有go程的调用栈踪迹都格式化写入到buf中。  
### 获取当前函数或者上层函数的标识号、文件名、调用方法在当前文件中的行号  
```go
func Caller(skip int) (pc uintptr, file string, line int, ok bool)

eg:
func main() {
  pc,file,line,ok := runtime.Caller(0)
  fmt.Println(pc)
  fmt.Println(file)
  fmt.Println(line)
  fmt.Println(ok)
}
```

### 获取与当前堆栈记录相关链的调用栈踪迹  
```go
func Callers(skip int, pc []uintptr) int
```
函数把当前go程调用栈上的调用栈标识符填入切片pc中，返回写入到pc中的项数。实参skip为开始在pc中记录之前所要跳过的栈帧数，0表示Callers自身的调用栈，1表示Callers所在的调用栈。返回写入p的项数。  

### 获取一个标识调用栈标识符pc对应的调用栈  
```go
func FuncForPC(pc uintptr) *Func
```
```go
func (f *Func) Name() string    // 获取调用栈所调用的函数的名字
func (f *Func) FileLine(pc uintptr) (file string, line int)     
// 获取调用栈所调用的函数的所在的源文件名和行号
func (f *Func) Entry() uintptr  // 获取该调用栈的调用栈标识符
```

### 获取当前进程执行的cgo调用次数  
```go
func NumCgoCall() int64
```

### 获取当前存在的go协程数  
```go
func NumGoroutine() int
```

### 终止掉当前的go协程  
```go
func Goexit()
```
Goexit终止调用它的go协程,其他协程不受影响,Goexit会在终止该go协程前执行所有的defer函数，前提是defer必须在它前面定义,如果在main go协程调用本方法,会终止该go协程,但不会让main返回,因为main函数没有返回,程序会继续执行其他go协程,当其他go协程执行完毕后,程序就会崩溃.  

### 让其他go协程优先执行,等其他协程执行完后,在执行当前的协程  
```go
func Gosched()
```

### 获取活跃的go协程的堆栈profile以及记录个数  
```go
func GoroutineProfile(p []StackRecord) (n int, ok bool)
```

### 将调用的go协程绑定到当前所在的操作系统线程，其它go协程不能进入该线程  
```go
func LockOSThread()  
```
将调用的go程绑定到它当前所在的操作系统线程。除非调用的go程退出或调用UnlockOSThread，否则它将总是在该线程中执行，而其它go程则不能进入该线程  
```go
func UnlockOSThread()
// 解除go协程与操作系统线程的绑定关系
```
将调用的go程解除和它绑定的操作系统线程。若调用的go程未调用LockOSThread，UnlockOSThread不做操作  

### 获取线程创建profile中的记录个数  
```go
func ThreadCreateProfile(p []StackRecord) (n int, ok bool)
```
返回线程创建profile中的记录个数。如果len(p)>=n，本函数就会将profile中的记录复制到p中并返回(n, true)。若len(p)< n，则不会更改p，而只返回(n, false)。  

绝大多数使用者应当使用runtime/pprof包，而非直接调用ThreadCreateProfile。  

### 控制阻塞profile记录go协程阻塞事件的采样率  
```go
func SetBlockProfileRate(rate int)
```
SetBlockProfileRate控制阻塞profile记录go程阻塞事件的采样频率。对于一个阻塞事件，平均每阻塞rate纳秒，阻塞profile记录器就采集一份样本。  

要在profile中包括每一个阻塞事件，需传入rate=1；要完全关闭阻塞profile的记录，需传入rate<=0。  

### 返回当前阻塞profile中的记录个数  
```go
func BlockProfile(p []BlockProfileRecord) (n int, ok bool)
```
BlockProfile返回当前阻塞profile中的记录个数。如果len(p)>=n，本函数就会将此profile中的记录复制到p中并返回(n, true)。如果len(p)< ，本函数则不会修改p，而只返回(n, false)。  

绝大多数使用者应当使用runtime/pprof包或testing包的-test.blockprofile标记， 而非直接调用 BlockProfile。  

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

# `sync` 包  

## 临时对象池  

Pool 用于存储临时对象，它将使用完毕的对象存入对象池中，在需要的时候取出来重复使用，目的是为了避免重复创建相同的对象造成 GC 负担过重。其中存放的临时对象随时可能被 GC 回收掉（如果该对象不再被其它变量引用）。  

从 Pool 中取出对象时，如果 Pool 中没有对象，将返回 nil，但是如果给 Pool.New 字段指定了一个函数的话，Pool 将使用该函数创建一个新对象返回。  

Pool 可以安全的在多个例程中并行使用，但 Pool 并不适用于所有空闲对象，Pool 应该用来管理并发的例程共享的临时对象，而不应该管理短寿命对象中的临时对象，因为这种情况下内存不能很好的分配，这些短寿命对象应该自己实现空闲列表。  

Pool 在开始使用之后，不能再被复制。  

```go
type Pool struct {
    // 创建临时对象的函数
    New func() interface{}
}

// 向临时对象池中存入对象
func (p *Pool) Put(x interface{})

// 从临时对象池中取出对象
func (p *Pool) Get() interface{}
```

## Once  

Once 的作用是多次调用但只执行一次，Once 只有一个方法，Once.Do()，向 Do 传入一个函数，这个函数在第一次执行 Once.Do() 的时候会被调用，以后再执行 Once.Do() 将没有任何动作，即使传入了其它的函数，也不会被执行，如果要执行其它函数，需要重新创建一个 Once 对象。  

Once 可以安全的在多个例程中并行使用。  

多次调用仅执行一次指定的函数 f：  
```go
func (o *Once) Do(f func())
```

eg：  
```go
func main() {
    var once sync.Once
    onceBody := func() {
        fmt.Println("Only once")
    }
    done := make(chan bool)
    for i := 0; i < 10; i++ {
        go func() {
            once.Do(onceBody) // 多次调用只执行一次
            done <- true
        }()
    }
    for i := 0; i < 10; i++ {
        <-done
    }
}

// 输出结果：
// Only once
```

## 互斥锁  

互斥锁用来保证在任一时刻，只能有一个例程访问某对象。Mutex 的初始值为解锁状态。Mutex 通常作为其它结构体的匿名字段使用，使该结构体具有 Lock 和 Unlock 方法。  

Mutex 可以安全的在多个例程中并行使用。

```go
// Locker 接口包装了基本的 Lock 和 UnLock 方法，用于加锁和解锁。
type Locker interface {
    Lock()
    Unlock()
}

// Lock 用于锁住 m，如果 m 已经被加锁，则 Lock 将被阻塞，直到 m 被解锁。
func (m *Mutex) Lock()

// Unlock 用于解锁 m，如果 m 未加锁，则该操作会引发 panic。
func (m *Mutex) Unlock()
```

eg：  
```go
type SafeInt struct {
    sync.Mutex
    Num int
}

func main() {
    count := SafeInt{}
    done := make(chan bool)
    for i := 0; i < 10; i++ {
        go func(i int) {
            count.Lock() // 加锁，防止其它例程修改 count
            count.Num += i
            fmt.Print(count.Num, " ")
            count.Unlock() // 修改完毕，解锁
            done <- true
        }(i)
    }
    for i := 0; i < 10; i++ {
        <-done
    }
}

// 输出结果（不固定）：
// 2 11 14 18 23 29 36 44 45 45
```

## 读写互斥锁  

RWMutex 比 Mutex 多了一个“读锁定”和“读解锁”，可以让多个例程同时读取某对象。RWMutex 的初始值为解锁状态。RWMutex 通常作为其它结构体的匿名字段使用。  

Mutex 可以安全的在多个例程中并行使用。  

```go
// Lock 将 rw 设置为写锁定状态，禁止其他例程读取或写入。
func (rw *RWMutex) Lock()

// Unlock 解除 rw 的写锁定状态，如果 rw 未被写锁定，则该操作会引发 panic。
func (rw *RWMutex) Unlock()

// RLock 将 rw 设置为读锁定状态，禁止其他例程写入，但可以读取。
func (rw *RWMutex) RLock()

// Runlock 解除 rw 的读锁定状态，如果 rw 未被读锁顶，则该操作会引发 panic。
func (rw *RWMutex) RUnlock()

// RLocker 返回一个互斥锁，将 rw.RLock 和 rw.RUnlock 封装成了一个 Locker 接口。
func (rw *RWMutex) RLocker() Locker
```

## 组等待  

WaitGroup 用于等待一组例程的结束。主例程在创建每个子例程的时候先调用 Add 增加等待计数，每个子例程在结束时调用 Done 减少例程计数。之后，主例程通过 Wait 方法开始等待，直到计数器归零才继续执行。  

```go
// 计数器增加 delta，delta 可以是负数。
func (wg *WaitGroup) Add(delta int)

// 计数器减少 1
func (wg *WaitGroup) Done()

// 等待直到计数器归零。如果计数器小于 0，则该操作会引发 panic。
func (wg *WaitGroup) Wait()
```

eg：  

```go
func main() {
    wg := sync.WaitGroup{}
    wg.Add(10)
    for i := 0; i < 10; i++ {
        go func(i int) {
            defer wg.Done()
            fmt.Print(i, " ")
        }(i)
    }
    wg.Wait()
}
// 输出结果（不固定）：
// 9 3 4 5 6 7 8 0 1 2
```

## 条件等待  

条件等待通过 Wait 让例程等待，通过 Signal 让一个等待的例程继续，通过 Broadcast 让所有等待的例程继续。  

在 Wait 之前应当手动为 c.L 上锁，Wait 结束后手动解锁。为避免虚假唤醒，需要将 Wait 放到一个条件判断循环中。官方要求的写法如下：  

```go
c.L.Lock()
for !condition() {
    c.Wait()
}
// 执行条件满足之后的动作...
c.L.Unlock()
```

Cond 在开始使用之后，不能再被复制。  

```go
type Cond struct {
    L Locker // 在“检查条件”或“更改条件”时 L 应该锁定。
} 

// 创建一个条件等待
func NewCond(l Locker) *Cond

// Broadcast 唤醒所有等待的 Wait，建议在“更改条件”时锁定 c.L，更改完毕再解锁。
func (c *Cond) Broadcast()

// Signal 唤醒一个等待的 Wait，建议在“更改条件”时锁定 c.L，更改完毕再解锁。
func (c *Cond) Signal()

// Wait 会解锁 c.L 并进入等待状态，在被唤醒时，会重新锁定 c.L
func (c *Cond) Wait()
```

eg：  
```go
func main() {
    condition := false // 条件不满足
    var mu sync.Mutex
    cond := sync.NewCond(&mu)
    // 让例程去创造条件
    go func() {
        mu.Lock()
        condition = true // 更改条件
        cond.Signal()    // 发送通知：条件已经满足
        mu.Unlock()
    }()
    mu.Lock()
    // 检查条件是否满足，避免虚假通知，同时避免 Signal 提前于 Wait 执行。
    for !condition {
        // 等待条件满足的通知，如果收到虚假通知，则循环继续等待。
        cond.Wait() // 等待时 mu 处于解锁状态，唤醒时重新锁定。
    }
    fmt.Println("条件满足，开始后续动作...")
    mu.Unlock()
}

// 输出结果：
// 条件满足，开始后续动作...
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

# `unicode` 包  

## 常量  
```go
const (
    MaxRune         = '\U0010FFFF' // Unicode 码点的最大值
    ReplacementChar = '\uFFFD'     // 表示无效的码点
    MaxASCII        = '\u007F'     // 最大 ASCII 值
    MaxLatin1       = '\u00FF'     // 最大 Latin-1 值
)
```

## RangeTable   
```go
// 判断字符 r 是否在 rangtab 范围内。
// 可用的 RangeTable 参见 go/src/unicode/tables.go。
func Is(rangeTab *RangeTable, r rune) bool

// RangeTable 定义一个 Unicode 码点集合，包含 16 位和 32 位两个范围列表。
// 这两个列表必须经过排序而且不能重叠。R32 中只能包含大于 16 位的值。
type RangeTable struct {
    R16         []Range16
    R32         []Range32
    LatinOffset int // R16 中 Hi <= MaxLatin1 的条目数。
}

// Range16 表示一个 16 位的 Unicode 码点范围。范围从 Lo 到 Hi，具有指定的步长。
type Range16 struct {
    Lo     uint16
    Hi     uint16
    Stride uint16 // 步长
}

// Range32 表示一个 32 位的 Unicode 码点范围。范围从 Lo 到 Hi，具有指定的步长。
// Lo 和 Hi 必须都大于 16 位。
type Range32 struct {
    Lo     uint32
    Hi     uint32
    Stride uint32 // 步长
}
```

## 判断及转换字符大小写、Title  
```go
// 判断字符 r 是否为大写格式
func IsUpper(r rune) bool

// 判断字符 r 是否为小写格式
func IsLower(r rune) bool

// 判断字符 r 是否为 Unicode 规定的 Title 字符
// 大部分字符的 Title 格式就是其大写格式
// 只有少数字符的 Title 格式是特殊字符
// 这里判断的就是特殊字符
func IsTitle(r rune) bool

// ToUpper 将字符 r 转换为大写格式
func ToUpper(r rune) rune

// ToLower 将字符 r 转换为小写格式
func ToLower(r rune) rune

// ToTitle 将字符 r 转换为 Title 格式
// 大部分字符的 Title 格式就是其大写格式
// 只有少数字符的 Title 格式是特殊字符
func ToTitle(r rune) rune

// To 将字符 r 转换为指定的格式
// _case 取值：UpperCase、LowerCase、TitleCase
func To(_case int, r rune) rune
```

示例1：判断汉字：  

```go
func main() {
    for _, r := range "Hello 世界！" {
        // 判断字符是否为汉字
        if unicode.Is(unicode.Scripts["Han"], r) {
            fmt.Printf("%c", r) // 世界
        }
    }
}
// 更多 unicode.Scripts 取值请参考：http://www.cnblogs.com/golove/p/3269099.html
```

示例2：判断大小写：  

```go
func main() {
    for _, r := range "Hello ＡＢＣ！" {
        // 判断字符是否为大写
        if unicode.IsUpper(r) {
            fmt.Printf("%c", r) // HＡＢＣ
        }
    }
    for _, r := range "Hello ａｂｃ！" {
        // 判断字符是否为小写
        if unicode.IsLower(r) {
            fmt.Printf("%c", r) // elloａｂｃ
        }
    }
    for _, r := range "Hello ᾏᾟᾯ！" {
        // 判断字符是否为标题
        if unicode.IsTitle(r) {
            fmt.Printf("%c", r) // ᾏᾟᾯ
        }
    }
}
```

示例3：输出 Unicode 规定的标题字符  

```go
func main() {
    for _, cr := range unicode.Lt.R16 {
        for i := cr.Lo; i <= cr.Hi; i += cr.Stride {
            fmt.Printf("%c", i)
        }
    } // ǅǈǋǲᾈᾉᾊᾋᾌᾍᾎᾏᾘᾙᾚᾛᾜᾝᾞᾟᾨᾩᾪᾫᾬᾭᾮᾯᾼῌῼ
}
```

示例4：转换大小写  

```go
func main() {
    s := "Hello 世界！"

    for _, r := range s {
        fmt.Printf("%c", unicode.ToUpper(r))
    } // HELLO 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.ToLower(r))
    } // hello 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.ToTitle(r))
    } // HELLO 世界！

    for _, r := range s {
        fmt.Printf("%c", unicode.To(unicode.UpperCase, r))
    } // HELLO 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.To(unicode.LowerCase, r))
    } // hello 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.To(unicode.TitleCase, r))
    } // HELLO 世界！
}
```

## rune操作  
```go
// ToUpper 将 r 转换为大写格式
// 优先使用指定的映射表 special
// 可用的 SpecialCase 参见 go/src/unicode/casetables.go。
func (special SpecialCase) ToUpper(r rune) rune

// ToLower 将 r 转换为小写格式
// 优先使用指定的映射表 special
func (special SpecialCase) ToLower(r rune) rune

// ToTitle 将 r 转换为 Title 格式
// 优先使用指定的映射表 special
func (special SpecialCase) ToTitle(r rune) rune

// SpecialCase 表示特定语言的大小写映射，比如土耳其语。
// SpecialCase 的方法可以自定义标准映射（通过重写）。
type SpecialCase []CaseRange

// CaseRange 表示一个简单的 Unicode 码点范围，用于大小写转换。
// 在 Lo 和 Hi 范围内的码点，如果要转换成大写，只需要加上 d[0] 即可
// 如果要转换为小写，只需要加上 d[1] 即可，如果要转换为 Title 格式，
// 只需要加上 d[2] 即可。
type CaseRange struct {
    Lo    uint32
    Hi    uint32
    Delta d // [3]rune
}

// CaseRanges 中 Delta 数组的索引。
const (
    UpperCase = iota
    LowerCase
    TitleCase
    MaxCase
)

// 如果一个 CaseRange 中的 Delta 元素是 UpperLower，则表示这个 CaseRange 是
// 一个有着连续的大写小写大写小写的范围。也就是说，Lo 是大写，Lo+1 是小写，
// Lo+2 是大写，Lo+3 是小写 ... 一直到 Hi 为止。
const (
    UpperLower = MaxRune + 1 // 不是一个有效的 Delta 元素
)
```

eg.
```go
func main() {
    s := "Hello 世界！"
    for _, r := range s {
        fmt.Printf("%c", unicode.SpecialCase(unicode.CaseRanges).ToUpper(r))
    } // HELLO 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.SpecialCase(unicode.CaseRanges).ToLower(r))
    } // hello 世界！
    for _, r := range s {
        fmt.Printf("%c", unicode.SpecialCase(unicode.CaseRanges).ToTitle(r))
    } // HELLO 世界！
}
```

## SimpleFold 环绕查找  

```go
// SimpleFold 在 Unicode 字符表中从字符 r 开始环绕查找（到尾部后再从头开始）
// 下一个与 r 大小写相匹配的字符（一个字符的大写、小写、标题三者视为大小写相
// 匹配），这个函数遵循 Unicode 定义的大小写环绕匹配表。
//
// 例如：
// SimpleFold('A') = 'a'
// SimpleFold('a') = 'A'
//
// SimpleFold('K') = 'k'
// SimpleFold('k') = 'K' (开尔文符号)
// SimpleFold('K') = 'K'
//
// SimpleFold('1') = '1'
func SimpleFold(r rune) rune
```

eg.
```go
func main() {
    s := "ΦφϕkKK"
    // 看看 s 里面是什么
    for _, c := range s {
        fmt.Printf("%x  ", c)
    }
    fmt.Println()
    // 大写，小写，标题 | 当前字符 -> 下一个匹配字符
    for _, v := range s {
        fmt.Printf("%c, %c, %c | %c -> %c\n",
            unicode.ToUpper(v),
            unicode.ToLower(v),
            unicode.ToTitle(v),
            v,
            unicode.SimpleFold(v),
        )
    }
}

// 输出结果：
// 3a6  3c6  3d5  6b  4b  212a
// Φ, φ, Φ | Φ -> φ
// Φ, φ, Φ | φ -> ϕ
// Φ, ϕ, Φ | ϕ -> Φ
// K, k, K | k -> K
// K, k, K | K -> k
// K, k, K | K -> K
```

## 判断  
```go
// IsDigit 判断 r 是否为一个十进制的数字字符
func IsDigit(r rune) bool

// IsNumber 判断 r 是否为一个数字字符 (类别 N)
func IsNumber(r rune) bool

// IsLetter 判断 r 是否为一个字母字符 (类别 L)
// 汉字也是一个字母字符
func IsLetter(r rune) bool

// IsSpace 判断 r 是否为一个空白字符
// 在 Latin-1 字符集中，空白字符为：\t, \n, \v, \f, \r,
// 空格, U+0085 (NEL), U+00A0 (NBSP)
// 其它空白字符的定义有“类别 Z”和“Pattern_White_Space 属性”
func IsSpace(r rune) bool

// IsControl 判断 r 是否为一个控制字符
// Unicode 类别 C 包含更多字符，比如代理字符
// 使用 Is(C, r) 来测试它们
func IsControl(r rune) bool

// IsGraphic 判断字符 r 是否为一个“图形字符”
// “图形字符”包括字母、标记、数字、标点、符号、空格
// 他们分别对应于 L、M、N、P、S、Zs 类别
// 这些类别是 RangeTable 类型，存储了相应类别的字符范围
func IsGraphic(r rune) bool

// IsPrint 判断字符 r 是否为 Go 所定义的“可打印字符”
// “可打印字符”包括字母、标记、数字、标点、符号和 ASCII 空格
// 他们分别对应于 L, M, N, P, S 类别和 ASCII 空格
// “可打印字符”和“图形字符”基本是相同的，不同之处在于
// “可打印字符”只包含 Zs 类别中的 ASCII 空格（U+0020）
func IsPrint(r rune) bool

// IsPunct 判断 r 是否为一个标点字符 (类别 P)
func IsPunct(r rune) bool

// IsSymbol 判断 r 是否为一个符号字符
func IsSymbol(r rune) bool

// IsMark 判断 r 是否为一个 mark 字符 (类别 M)
func IsMark(r rune) bool

// IsOneOf 判断 r 是否在 set 范围内
func IsOneOf(set []*RangeTable, r rune) bool
```

eg.
```go
func main() {
    fmt.Println() // 数字
    for _, r := range "Hello 123１２３一二三！" {
        if unicode.IsDigit(r) {
            fmt.Printf("%c", r)
        }
    } // 123１２３

    fmt.Println() // 数字
    for _, r := range "Hello 123１２３一二三！" {
        if unicode.IsNumber(r) {
            fmt.Printf("%c", r)
        }
    } // 123１２３

    fmt.Println() // 字母
    for _, r := range "Hello\n\t世界！" {
        if unicode.IsLetter(r) {
            fmt.Printf("%c", r)
        }
    } // Hello世界

    fmt.Println() // 空白
    for _, r := range "Hello \t世　界！\n" {
        if unicode.IsSpace(r) {
            fmt.Printf("%q", r)
        }
    } // ' ''\t''\u3000''\n'

    fmt.Println() // 控制字符
    for _, r := range "Hello\n\t世界！" {
        if unicode.IsControl(r) {
            fmt.Printf("%#q", r)
        }
    } // '\n''\t'

    fmt.Println() // 可打印
    for _, r := range "Hello　世界！\t" {
        if unicode.IsPrint(r) {
            fmt.Printf("%c", r)
        }
    } // Hello世界！

    fmt.Println() // 图形
    for _, r := range "Hello　世界！\t" {
        if unicode.IsGraphic(r) {
            fmt.Printf("%c", r)
        }
    } // Hello　世界！

    fmt.Println() // 掩码
    for _, r := range "Hello ៉៊់៌៍！" {
        if unicode.IsMark(r) {
            fmt.Printf("%c", r)
        }
    } // ៉៊់៌៍

    fmt.Println() // 标点
    for _, r := range "Hello 世界！" {
        if unicode.IsPunct(r) {
            fmt.Printf("%c", r)
        }
    } // ！

    fmt.Println() // 符号
    for _, r := range "Hello (<世=界>)" {
        if unicode.IsSymbol(r) {
            fmt.Printf("%c", r)
        }
    } // <=>
}
```

eg.判断汉字和标点：  

```go
func main() {
    // 将 set 设置为“汉字、标点符号”
    set := []*unicode.RangeTable{unicode.Han, unicode.P}
    for _, r := range "Hello 世界！" {
        if unicode.IsOneOf(set, r) {
            fmt.Printf("%c", r)
        }
    } // 世界！
}
```

eg.输出所有 mark 字符：  
```go
func main() {
    for _, cr := range unicode.M.R16 {
        Lo, Hi, Stride := rune(cr.Lo), rune(cr.Hi), rune(cr.Stride)
        for i := Lo; i >= Lo && i <= Hi; i += Stride {
            if unicode.IsMark(i) {
                fmt.Printf("%c", i)
            }
        }
    }
}
```

---

# `unicode/utf16` 包  

```go
// IsSurrogate 判断 r 是否为代理区字符
// 两个代理区字符可以用来组合成一个 utf16 编码
func IsSurrogate(r rune) bool

// EncodeRune 将字符 r 编码成 UTF-16 代理对
// r：要编码的字符
// 如果 r < 0x10000 ，则无需编码，其 UTF-16 序列就是其自身
// r1：编码后的 UTF-16 代理对的高位码元
// r2：编码后的 UTF-16 代理对的低位码元
// 如果 r 不是有效的 Unicode 字符，或者是代理区字符，或者无需编码
// 则返回 U+FFFD, U+FFFD
func EncodeRune(r rune) (r1, r2 rune)

// DecodeRune 将 UTF-16 代理对解码成一个 Unicode 字符
// r1：是 UTF-16 代理对的高位码元
// r2：是 UTF-16 代理对的低位码元
// 返回值为解码后的 Unicode 字符
// 如果 r1 或 r2 不是有效的 UTF-16 代理区字符，则返回 U+FFFD
func DecodeRune(r1, r2 rune) rune

// Decode 将 UTF-16 序列 s 解码成 Unicode 字符序列并返回
func Decode(s []uint16) []rune

// Encode 将 s 编码成 UTF-16 序列并返回
func Encode(s []rune) []uint16
```

eg.
```go
func main() {
    fmt.Printf("%t, ", utf16.IsSurrogate(0xD400)) // false
    fmt.Printf("%t, ", utf16.IsSurrogate(0xDC00)) // true
    fmt.Printf("%t\n", utf16.IsSurrogate(0xDFFF)) // true

    r1, r2 := utf16.EncodeRune('𠀾')
    fmt.Printf("%x, %x\n", r1, r2) // d840, dc3e

    r := utf16.DecodeRune(0xD840, 0xDC3E)
    fmt.Printf("%c\n", r) // d840, dc3e

    u := []uint16{'不', '会', 0xD840, 0xDC3E}
    s := utf16.Decode(u)
    fmt.Printf("%c", s) // [不 会 𠀾]
}
```

---

# `unicode/utf8` 包  

## 常量  
```go
// 编码所需的基本数字
const (
    RuneError = '\uFFFD'     // 错误的 Rune 或 Unicode 代理字符
    RuneSelf  = 0x80         // ASCII 字符范围
    MaxRune   = '\U0010FFFF' // Unicode 码点的最大值
    UTFMax    = 4            // 一个字符编码的最大长度
)
```

## 函数  
```go
// 将 r 转换为 UTF-8 编码写入 p 中（p 必须足够长，通常为 4 个字节）
// 如果 r 是无效的 Unicode 字符，则写入 RuneError
// 返回写入的字节数
func EncodeRune(p []byte, r rune) int

// 解码 p 中的第一个字符，返回解码后的字符和 p 中被解码的字节数
// 如果 p 为空，则返回（RuneError, 0）
// 如果 p 中的编码无效，则返回（RuneError, 1）
// 无效编码：UTF-8 编码不正确（比如长度不够）、结果超出 Unicode 范围、编码不是最短的。
// 关于最短编码：可以用四个字节编码一个单字节字符，但它不是最短的，比如：
// [111100000 10000000 10000000 10111000] 不是最短的，应该使用 [00111000]
func DecodeRune(p []byte) (r rune, size int)

// 功能同上，参数为字符串
func DecodeRuneInString(s string) (r rune, size int)

// 解码 p 中的最后一个字符，返回解码后的字符，和 p 中被解码的字节数
// 如果 p 为空，则返回（RuneError, 0）
// 如果 p 中的编码无效，则返回（RuneError, 1）
func DecodeLastRune(p []byte) (r rune, size int)

// 功能同上，参数为字符串
func DecodeLastRuneInString(s string) (r rune, size int)

// FullRune 检测 p 中第一个字符的 UTF-8 编码是否完整（完整并不表示有效）。
// 一个无效的编码也被认为是完整字符，因为它将被转换为一个 RuneError 字符。
// 只有“编码有效但长度不够”的字符才被认为是不完整字符。
// 也就是说，只有截去一个有效字符的一个或多个尾部字节，该字符才算是不完整字符。
// 举例：
// "好"     是完整字符
// "好"[1:] 是完整字符（首字节无效，可转换为 RuneError 字符）
// "好"[2:] 是完整字符（首字节无效，可转换为 RuneError 字符）
// "好"[:2] 是不完整字符（编码有效但长度不够）
// "好"[:1] 是不完整字符（编码有效但长度不够）
func FullRune(p []byte) bool

// 功能同上，参数为字符串
func FullRuneInString(s string) bool

// 返回 p 中的字符个数
// 错误的 UTF8 编码和长度不足的 UTF8 编码将被当作单字节的 RuneError 处理
func RuneCount(p []byte) int

// 功能同上，参数为字符串
func RuneCountInString(s string) (n int)

// RuneLen 返回需要多少字节来编码字符 r，如果 r 是无效的字符，则返回 -1
func RuneLen(r rune) int

// 判断 b 是否为 UTF8 字符的首字节编码，最高位(bit)是不是 10 的字节就是首字节。
func RuneStart(b byte) bool

// Valid 判断 p 是否为完整有效的 UTF8 编码序列。
func Valid(p []byte) bool

// 功能同上，参数为字符串
func ValidString(s string) bool

// ValidRune 判断 r 能否被正确的转换为 UTF8 编码
// 超出 Unicode 范围的码点或 UTF-16 代理区中的码点是不能转换的
func ValidRune(r rune) bool
```

eg.

```go
func main() {
    b := make([]byte, utf8.UTFMax)

    n := utf8.EncodeRune(b, '好')
    fmt.Printf("%v：%v\n", b, n) // [229 165 189 0]：3

    r, n := utf8.DecodeRune(b)
    fmt.Printf("%c：%v\n", r, n) // 好：3

    s := "大家好"
    for i := 0; i < len(s); {
        r, n = utf8.DecodeRuneInString(s[i:])
        fmt.Printf("%c：%v   ", r, n) // 大：3   家：3   好：3
        i += n
    }
    fmt.Println()

    for i := len(s); i > 0; {
        r, n = utf8.DecodeLastRuneInString(s[:i])
        fmt.Printf("%c：%v   ", r, n) // 好：3   家：3   大：3
        i -= n
    }
    fmt.Println()

    b = []byte("好")
    fmt.Printf("%t, ", utf8.FullRune(b))     // true
    fmt.Printf("%t, ", utf8.FullRune(b[1:])) // true
    fmt.Printf("%t, ", utf8.FullRune(b[2:])) // true
    fmt.Printf("%t, ", utf8.FullRune(b[:2])) // false
    fmt.Printf("%t\n", utf8.FullRune(b[:1])) // false

    b = []byte("大家好")
    fmt.Println(utf8.RuneCount(b)) // 3

    fmt.Printf("%d, ", utf8.RuneLen('A'))          // 1
    fmt.Printf("%d, ", utf8.RuneLen('\u03A6'))     // 2
    fmt.Printf("%d, ", utf8.RuneLen('好'))          // 3
    fmt.Printf("%d, ", utf8.RuneLen('\U0010FFFF')) // 4
    fmt.Printf("%d\n", utf8.RuneLen(0x1FFFFFFF))   // -1

    fmt.Printf("%t, ", utf8.RuneStart("好"[0])) // true
    fmt.Printf("%t, ", utf8.RuneStart("好"[1])) // false
    fmt.Printf("%t\n", utf8.RuneStart("好"[2])) // false

    b = []byte("你好")
    fmt.Printf("%t, ", utf8.Valid(b))     // true
    fmt.Printf("%t, ", utf8.Valid(b[1:])) // false
    fmt.Printf("%t, ", utf8.Valid(b[2:])) // false
    fmt.Printf("%t, ", utf8.Valid(b[:2])) // false
    fmt.Printf("%t, ", utf8.Valid(b[:1])) // false
    fmt.Printf("%t\n", utf8.Valid(b[3:])) // true

    fmt.Printf("%t, ", utf8.ValidRune('好'))        // true
    fmt.Printf("%t, ", utf8.ValidRune(0))          // true
    fmt.Printf("%t, ", utf8.ValidRune(0xD800))     // false  代理区字符
    fmt.Printf("%t\n", utf8.ValidRune(0x10FFFFFF)) // false  超出范围
}
```


---
*`to be continued...`*  

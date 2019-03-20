---
title: Golang-pkg-Notes
date: 2019-03-20 22:16:19
tags: Go
categories: Go
permalink: Golang-pkg
---
# `strings` 包  
Go 中使用 `strings` 包对字符串进行操作。  
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
`ToLower` 讲字符串中的 Unicode 字符全部转换为相应的小写字符：
```go
    strings.Tolower(s) string
```
`ToUpper` 讲字符串中的 Unicode 字符全部转换为相应的大写字符：
```go
    strings.ToUpper(s) string
```
## 修剪字符串  
`strings.TrimSpace(s)` 用于剔除字符串开头和结尾的空白符号；  
`strings.Trim(s, "cut")` 用于剔除字符串开头和结尾的 `cut`；  
`TrimLeft` 或者 `TrimRight` 用于只剔除开头或结尾的字符串。  
## 分割字符串  
`strings.Fields(s)` 会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice。  
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

其他有关字符串操作的文档参考 [官方文档](http://golang.org/pkg/strings/) 或 [国内访问文档](http://docs.studygolang.com/pkg/strings/)。

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

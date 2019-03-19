---
title: Golang in Action Notes
date: 2019-03-19 16:01:21
tags: Go
categories: Go
permalink: Go-Action-Notes
---

# Tips  
## `,ok` 模式  
（1）在函数返回时检测错误  

```go
value, err := pack1.Func1(param1)

if err != nil {
    fmt.Printf(“Error %s in pack1.Func1 with parameter %v”, err.Error(), param1)
    return err
}

// 函数Func1没有错误:
Process(value)

e.g.: os.Open(file) strconv.Atoi(str)
```

这种模式也常用于通过`defer`使程序从`panic`中恢复执行。  

要实现简洁的错误检测代码，更好的方式是使用闭包，参考[第16.10.2小节](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/16.10.md)

（2）检测映射中是否存在一个键值：key1在映射map1中是否有值？[参考](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/08.2.md)  
```go
if value, ok := map1[key1]; ok {
    // ...
}
// key1不存在
…
```

（3）检测一个接口类型变量`varI`是否包含了类型`T`：类型断言 [参考](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/11.3.md)  
```go
if value, ok := varI.(T); ok {
    Process(value)
}
// 接口类型varI没有包含类型T
```

(4)检测一个通道 `ch` 是否关闭 [参考](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/14.3.md)  
```go
for input := range ch {
     Process(input)
}
```
或者：
```
for {
    if input, open := <-ch; !open {
        break // 通道是关闭的
    }
     Process(input)
}
```

## 出于性能考虑的使用代码片段  
### 字符串  
（1）如何修改字符串中的一个字符：  

```go
str:="hello"
c:=[]byte(str)
c[0]='c'
s2:= string(c) // s2 == "cello"
```

（2）如何获取字符串的子串：  

```go
substr := str[n:m]
```

（3）如何使用`for`或者`for-range`遍历一个字符串：  

```go
// gives only the bytes:
for i:=0; i < len(str); i++ {
… = str[i]
}
// gives the Unicode characters:
for ix, ch := range str {
…
}
```

（4）如何获取一个字符串的字节数：`len(str)`  

 如何获取一个字符串的字符数：  

 最快速：`utf8.RuneCountInString(str)`   

 `len([]rune(str))`   

（5）如何连接字符串：  

 最快速：
`with a bytes.Buffer`（参考[章节7.2](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/07.2.md)）

`Strings.Join()`（参考[章节4.7](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/04.7.md)）
    
使用`+=`：

 ```go
 str1 := "Hello " 
 str2 := "World!"
 str1 += str2 //str1 == "Hello World!"
 ```

（6）如何解析命令行参数：使用`os`或者`flag`包

（参考[例12.4](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/examples/chapter_12/fileinput.go)）  

### 数组 切片  
创建：  

`arr1 := new([len]type)`  

`slice1 := make([]type, len)`  

初始化：  

`arr1 := [...]type{i1, i2, i3, i4, i5}`  

`arrKeyValue := [len]type{i1: val1, i2: val2}`  

`var slice1 []type = arr1[start:end]`  

（1）如何截断数组或者切片的最后一个元素：  

`line = line[:len(line)-1]`  

（2）如何使用`for`或者`for-range`遍历一个数组（或者切片）：  

```go
for i:=0; i < len(arr); i++ {
… = arr[i]
}
for ix, value := range arr {
…
}
```

（3）如何在一个二维数组或者切片`arr2Dim`中查找一个指定值`V`：  

```go
found := false
Found: for row := range arr2Dim {
    for column := range arr2Dim[row] {
        if arr2Dim[row][column] == V{
            found = true
            break Found
        }
    }
}
```

### map  
创建：    `map1 := make(map[keytype]valuetype)`  

初始化：   `map1 := map[string]int{"one": 1, "two": 2}`  

（1）如何使用`for`或者`for-range`遍历一个映射：  

```go
for key, value := range map1 {
…
}
```

（2）如何在一个映射中检测键`key1`是否存在：`val1, isPresent = map1[key1]`  

返回值：键`key1`对应的值或者`0`, `true`或者`false`  
    
（3）如何在映射中删除一个键：`delete(map1, key1)`  

### 结构体  
创建：  

```go
type struct1 struct {
    field1 type1
    field2 type2
    …
}
ms := new(struct1)
```

初始化：  

```go
ms := &struct1{10, 15.5, "Chris"}
```

当结构体的命名以大写字母开头时，该结构体在包外可见。  
通常情况下，为每个结构体定义一个构建函数，并推荐使用构建函数初始化结构体（参考[例10.2](https://github.com/reitake/the-way-to-go_ZH_CN/blob/master/eBook/examples/chapter_10/person.go)）：  

```go    
ms := Newstruct1{10, 15.5, "Chris"}
func Newstruct1(n int, f float32, name string) *struct1 {
    return &struct1{n, f, name} 
}
```

### 接口  
（1）如何检测一个值`v`是否实现了接口`Stringer`：  

```go
if v, ok := v.(Stringer); ok {
    fmt.Printf("implements String(): %s\n", v.String())
}
```

（2）如何使用接口实现一个类型分类函数：  
    
```go
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
            fmt.Printf("param #%d is a bool\n", i)
        case float64:
            fmt.Printf("param #%d is a float64\n", i)
        case int, int64:
            fmt.Printf("param #%d is an int\n", i)
        case nil:
            fmt.Printf("param #%d is nil\n", i)
        case string:
            fmt.Printf("param #%d is a string\n", i)
        default:
            fmt.Printf("param #%d’s type is unknown\n", i)
        }
    }
}
```



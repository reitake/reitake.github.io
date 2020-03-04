---
title: Go の Command
tags: [Go, Note]
categories: Go
permalink: Go-Command
date: 2019-04-21 22:28:25
---
<center> <font color="#bababa">

***GO 命令***

</font></center>
<!--more-->

---

# 前言  

--> [官方的命令文档](https://golang.org/cmd/cgo/) <--  

## 查看可用命令  

直接在终端中输入 `go help` 即可显示所有的 go 命令以及相应命令功能的简介，主要有：  

- build: 编译包和依赖
- clean: 移除对象文件
- doc: 显示包或者符号的文档
- env: 打印go的环境信息
- bug: 启动错误报告
- fix: 运行go tool fix
- fmt: 运行gofmt进行格式化
- generate: 从processing source生成go文件
- get: 下载并安装包和依赖
- install: 编译并安装包和依赖
- list: 列出包
- run: 编译并运行go程序
- test: 运行测试
- tool: 运行go提供的工具
- version: 显示go的版本
- vet: 运行go tool vet

## 使用命令  

命令的使用方式：  `go command [args]` ，`go help <command>`  

在运行`go help`时，不仅仅打印了这些命令的基本信息，还给出了一些概念的帮助信息（`go help <topic>`）:  

+ c: Go和c的相互调用
+ buildmode: 构建模式的描述
+ filetype: 文件类型
+ gopath: GOPATH环境变量
+ environment: 环境变量
+ importpath: 导入路径语法
+ packages: 包列表的描述
+ testflag: 测试符号描述
+ testfunc: 测试函数描述

---

# `go build`  

类似其他静态，要执行 go 程序，需要先编译成可执行文件。  

`go build`：用来编译 go 程序生成可执行文件。  

+ 但不是所有 go 程序都可以编译生成可执行文件，要生成可执行文件，go 程序需要满足两个条件（反之则不会生成任何文件，只是做检查性编译）：
    * 该 go 程序需要属于 main 包
    * 在 main 包中必须还得包含 main() 函数

```go
$ go run hello.go   # 将会生成可执行文件 hello
$ ./hello           # 运行可执行文件
Hello World!
```

加一些标记：  

+ -v 标记，可以把命令执行过程中构建的包名（包含编译过程中依赖的包）打印出来，如果 go build 的是一个源码文件，则会打印出的包名为 command-line-arguments，这是编译源码文件时生成的虚拟包名，所以看到不用觉得奇怪。
+ -x 标记，可以打印编译期间所用到的所有 shell 命令。
+ -o 标记，用来指定生成文件的路径和名称。
+ -a 标记，强制重新编译。
+ -buildmode=shared 标记，这个参数可以指定当前编译生成的结果类型，如静态库和动态库。GO语言默认使用静态编译，好处是部署时非常简单，但使用动态库，可以减少分发包的大小，大家可以根据实际情况选择。注意，目前在windows下尚不支持编译成动态库。
+ 更多参数，请使用 go build -h 或 go help build 查看。



# `go run`  

`go run`：编译并运行，但不会产生中间文件

```go
$ go run hello.go
Hello World!
```

# `go clean`  

用于清除产生的可执行程序：  

```go
$ go clean    # 不加参数，可以删除当前目录下的所有可执行文件
$ go clean sourcefile.go  # 会删除对应的可执行文件
```

# `go fmt`  

格式化代码。注意不包含代码包中的子代码包。  

```go
$ go fmt main.go
```

更多信息，通过 `gofmt -h` 查看。  


# `go doc` & `godoc`  

用于快速查看包文档，`go doc package` 命令将在中断中打印出指定 package 的文档。  

另外，`godoc` 可以启动我们自己的文档服务器：  

```go
$ godoc -http=:8080
```

然后我们就可与在浏览器 `localhost:8080` 中查看go文档了。  

# `go get`  

用来安装第三方包，格式：`go get src`，从指定源上面下载或者更新指定的代码和依赖，并对他们进行编译和安装。  

eg.例如我们像使用 beego 来开发 web 应用，我们就需要获取 beego：

```go
$ go get github.com/astaxie/beego
```

这条命令将会自动下载安装 beego 以及它的依赖，然后我们就可以使用下面的方式使用:  

```go
package main

import "github.com/astaxie/beego"   # 这里需要使用 src 下的完整路径

func main() {
    beego.Run()
}
```

# `go install`  

用来编译和安装 go 程序。与 build 命令进行对比：  

|     |  install  |   build  |
|-----|-----------|----------|
|生成的可执行文件路径|工作目录下的bin目录下|当前目录下|
|可执行文件的名字    |与源码所在目录同名   |默认与源程序同名，可以使用-o选项指定|
|依赖  | 将依赖的保安放到工作目录下的pkg文件夹下|-|

# `go test`  

用来运行测试的命令，这种测试是以包围单位的。需要遵循一定的规则：

+ 测试源文件是名称以`_test.go`为后缀的
+ 测试源文件内包含若干测试函数的源码文件
+ 测试函数一般是以`Test`为名称前缀，并有一个类型为`testing.T`的参数

# `go list`  

不加任何标记直接使用，是显示指定包的导入路径，如 `go list net/http` 就显示 `net/http`。  

该命令加上 -json 标记可以显示完整信息。  

```go
$ go list -json time
```

如果只想显示指定信息，可以使用 -f 标记，如 go list -f { {.GoFiles} } net/http 可以显示 net/http 包中的 GO 源码文件列表。（所以可以理解，默认的 go list 相当于 go list -f { {.ImportPath} }）  

# `gofix`  

简单的说，这是一个当GO语言版本升级之后，把代码包中旧的语法更新成新版本语法的自动化工具。它是 go tool fix 的简单封装，它作用于代码包。  

当需要升级自己的项目或者升级下载的第三方代码包，可以使用此方法。（下载并升级代码包可以使用 go get -fix 命令 ）  

# `go vet`  

静态检查工具，一般项目快完成时候进行优化时需要。  

# `go tool pprof`  

性能检查工具...  

# `go env`  

用于打印GO语言的环境信息，如 GOPATH 是工作区目录，GOROOT 是GO语言安装目录，GOBIN 是通过 go install 命令生成可执行文件的存放目录（默认是当前工作区的 bin 目录下），GOEXE 为生成可执行文件的后缀。  

# 转成汇编代码  

```go
$ go tool objdump -s "operate\.Login" server
```

上面的意思是，解析可执行文件server，将其中的 operate 包的 Login 方法转成汇编代码。  

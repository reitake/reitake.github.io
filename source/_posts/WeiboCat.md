---
title: WeiboCat With Golang
tags: [Go, Go项目]
categories: [Go]
permalink: WeiboCat
date: 2019-04-15 09:52:23
---
<center> <font color="#bababa">***Jacky & Jason***</font><br/> </center>
<!--more-->
---

# 说明  
## 项目情况  

* 项目名：`WeiboCat`
* 想要实现的功能：每天定时在微博上发一张家里喵的图片
* 启动日期：2019年4月
* 项目代码地址：[GitHub-GoWeiboCat](https://github.com/reitake/GoWeiboCat)

## 项目框架  

- [ ] 微博登录  
- [ ] 微博发post
- [x] 本地图片文件夹管理
- [x] “闹钟”功能
- [ ] 用.json作为config

---

# 待后续优化内容  

---

# 版本 Change Log  

---

# 过程中踩到的坑

## movepic.go  
### 把图片改名成时间  

文件名不能包含`：`这个符号，否则用`os.Rename`改名不成功，提示错误：`The filename, directory name, or volume label syntax is incorrect.`。同样地，新文件名不能包含`*`、`\`、`/`等字符。  

正确的：  

```go
func renameImage(folderPath string, imageName string){
    oldpath := folderPath + PTHSEP + imageName
    fmt.Println("oldpath:", oldpath)
    newpath := folderPath + PTHSEP + time.Now().Format("2006-01-02-15-04-05") + ".png"
    fmt.Println("newpath:", newpath)
    err := os.Rename(oldpath, newpath)
    fmt.Println("err:", err)
    return
}
```

错误的：  

```go
……
newpath := folderPath + PTHSEP + time.Now().Format("2006-01-02-15:04:05") + ".png"
……
```

### 判断文件夹中是否有图片  
先用`ioutil.ReadDir(imagesFolderPath)`读取目录下文件，再根据拓展名判断是否是规定格式的图片，不是文件的剔除，剔除后，检查时候还存在剩余图片。  

此时应该用`len()`判断，而不是判断切片是否为`nil`。  

```go
    images, err := ioutil.ReadDir(imagesFolderPath)
    if err != nil {
        return err
    }

    for i := 0; i < len(images); {
        if strings.HasSuffix(strings.ToLower(images[i].Name()), ".jpg") || strings.HasSuffix(strings.ToLower(images[i].Name()), ".png") || strings.HasSuffix(strings.ToLower(images[i].Name()), ".jpeg") {
            i++
        } else {
            images = append(images[:i], images[i+1:]...)
        }
    }
    
    if len(images) == 0 {
        fmt.Printf("\"%s\"文件夹中未找到 .jpg/.png/.jpeg 后缀的图片！\n", IMAGESFOLDER)
        err = errors.New("图片库文件夹中未发现图片！")
        return err
    }
```
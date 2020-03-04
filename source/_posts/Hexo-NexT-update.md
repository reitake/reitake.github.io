---
title: 升级Hexo+NexT版本
tags:
  - Hexo
  - NexT
categories: 备忘
permalink: Hexo-NexT-update
date: 2020-03-04 16:01:33
---

<center> <font color="#bababa">

***为什么要手痒升级呢？***

</font></center>
<!--more-->

---

# Update Log

- 升级时间：2020年3月
    + `Hexo`版本：v3.8.0 → v4.2.0
    + `NexT`版本：v7.0.1 → v7.7.2

---

#  升级操作

## 升级思路

因为目前对于本Blog的管理是：网页托管于`github`仓库，部署在`master`分支，Hexo和NexT的网站后台操作内容放在同一个仓库的`Hexo.source`分支。

计划先在本地升级Hexo和NexT，做好source搬迁、config配置等工作；然后在仓库中新建一个空的分支，把本地的后台文件复制进去，然后提交这个分支，今后维护新的后台分支就可以了。

## 具体操作

### 本地升级Hexo

Hexo版本升级可以通过npm实现，相关命令如下：  

先全局升级`hexo-cli`：-g表示全局升级。`hexo`本身是一个静态博客生成工具，具备编译Markdown、拼接主题模板、生成 HTML、上传 Git 等基本功能。`hexo-cli`能够将这些功能封装为命令，提供给用户通过`hexo server / hexo deploy`等命令调用的模块。`CLI = Command Line Interface`命令行界面。


```
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```

### 创建空分支

现在Github上创建分支是基于其他分支的，没法直接创建空的。

需要在创建的分支后，  

```
$ git rm -rf .
```

不要漏了`.`

然后考进来，commit。

### 更新NexT

去[Github-NexT](https://github.com/theme-next/hexo-theme-next/releases)下载个新版，然后解压到`/theme/`下。

剩下的就慢慢调配置吧……


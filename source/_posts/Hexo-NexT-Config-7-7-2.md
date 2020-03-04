---
title: Hexo + NexT(v7.7.2) 博客配置备忘
tags:
  - Hexo
  - NexT
categories: 备忘
permalink: Hexo-NexT-Config-7-7-2
date: 2020-03-04 18:20:40
---

<center> <font color="#bababa">

***好好的为何要折腾自己？***

</font></center>
<!--more-->

---

# 配置历史

- [Hexo + NexT(v7.1.0) 博客配置备忘](../../../../2019/03/27/Hexo-NexT-Config/)

---

# 基本信息  
建站驱动：`Hexo` v4.2.0  

主题： `NexT`.`Pisces` v7.7.2  

站点配置文件：`/_config.yml`  

主题配置文件：`/themes/next7.7.2/_config.yml`  

---

# 与之前配置的方法有变动的

## 不显示文章信息中“评论数”三个字

现在版本显示的是`Valine：`，改了下`/themes/next/layout/_macro/post.swig`也暂时没找到删去的办法，先放着罢。

## 增加页脚显示网站运营时间功能

现在不需要去在`/themes/next/layout/_parrials/footer.swig`末尾新增内容了，现在NexT支持自定义风格。  

在`主题配置文件`中，开启自定义功能：  

```go
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.swig
  #header: source/_data/header.swig
  #sidebar: source/_data/sidebar.swig
  #postMeta: source/_data/post-meta.swig
  #postBodyEnd: source/_data/post-body-end.swig
  footer: source/_data/footer.swig
  #bodyEnd: source/_data/body-end.swig
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```

然后去网站根目录的`/source/`下新建文件夹`_data`和文件`footer.swig`，在里面加入自定义的代码：

```html
<span class="post-meta-item-icon">
    <i class="fa fa-clock-o"></i>
</span>
<span id="sitetime"></span>
<script language="javascript">
  function siteTime(){
    window.setTimeout("siteTime()", 1000);
    var seconds = 1000;
    var minutes = seconds * 60;
    var hours = minutes * 60;
    var days = hours * 24;
    var years = days * 365;
    var today = new Date();
    var todayYear = today.getFullYear();
    var todayMonth = today.getMonth()+1;
    var todayDate = today.getDate();
    var todayHour = today.getHours();
    var todayMinute = today.getMinutes();
    var todaySecond = today.getSeconds();
    /* Date.UTC() -- 返回date对象距世界标准时间(UTC)1970年1月1日午夜之间的毫秒数(时间戳)
    year - 作为date对象的年份，为4位年份值
    month - 0-11之间的整数，做为date对象的月份
    day - 1-31之间的整数，做为date对象的天数
    hours - 0(午夜24点)-23之间的整数，做为date对象的小时数
    minutes - 0-59之间的整数，做为date对象的分钟数
    seconds - 0-59之间的整数，做为date对象的秒数
    microseconds - 0-999之间的整数，做为date对象的毫秒数 */
    var t1 = Date.UTC(2019,03,07,10,00,00); //建站时间
    var t2 = Date.UTC(todayYear,todayMonth,todayDate,todayHour,todayMinute,todaySecond);
    var diff = t2-t1;
    var diffYears = Math.floor(diff/years);
    var diffDays = Math.floor((diff/days)-diffYears*365);
    var diffHours = Math.floor((diff-(diffYears*365+diffDays)*days)/hours);
    var diffMinutes = Math.floor((diff-(diffYears*365+diffDays)*days-diffHours*hours)/minutes);
    var diffSeconds = Math.floor((diff-(diffYears*365+diffDays)*days-diffHours*hours-diffMinutes*minutes)/seconds);
    document.getElementById("sitetime").innerHTML=" 已运行"+diffYears+" 年 "+diffDays+" 天 "+diffHours+" 小时 "+diffMinutes+" 分钟 "+diffSeconds+" 秒 ~喵~";
  }/*建站不到1年，不想显示“0”年可以把year部分注释掉*/
  siteTime();
</script>
```

## 增加文章边框阴影

与上面类似，在`站点配置文件中`开启:  

```go
  style: source/_data/styles.styl
```

然后在根目录下建立`/source/_data/styles.styl`：

```css
// Custom styles.
// 主页文章添加阴影效果
.post-block {
   margin-top: 0px;
   margin-bottom: 24px;
   padding: 35px;
   -webkit-box-shadow: 0 0 24px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```

## 修改新建文章模板

`/scaffolds/post.md`中修改内容：

```
---
title: {{ title }}
date: {{ date }}
tags: [tag1, tag2]
categories: 分类名
permalink:      // 文章自定义链接
---

<center> <font color="#bababa">

***首页展示的字***

</font></center>
<!--more-->

---
```

（写markdown的时候多加几个回车能避免很多问题……）

## 修改文章末尾 tags 的符号（`#`改成图标）

不用去改`/themes/next/layout/_macro/post.swig`了，新版NexT意见在`主题配置文件`中集成了这个配置：

```go
# Use icon instead of the symbol # to indicate the tag at the bottom of the post
tag_icon: true
```
## 调整网站字体大小  

新版的字体偏大，可以去`站点配置文件`中`enable`字体功能：  

```go
font:
  enable: true

  # Uri of fonts host, e.g. https://fonts.googleapis.com (Default).
  host:

  # Font options:
  # `external: true` will load this font family from `host` above.
  # `family: Times New Roman`. Without any quotes.
  # `size: x.x`. Use `em` as unit. Default: 1 (16px)

  # Global font settings used for all elements inside <body>.
  global:
    external: true
    family: Lato
    size: 0.875

  # Font settings for site title (.site-title).
  title:
    external: true
    family:
    size:

  # Font settings for headlines (<h1> to <h6>).
  headings:
    external: true
    family:
    size:

  # Font settings for posts (.post-body).
  posts:
    external: true
    family:

  # Font settings for <code> and code blocks.
  codes:
    external: true
    family: 
```

---

# 安装插件  

站点地图：  

```
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save     //可选
```

站内搜索（Local Search）：  

```
$ npm install hexo-generator-search
$ npm install hexo-generator-searchdb
```


部署工具：  

```
$ npm install hexo-deployer-git --save
```

字数统计、文章阅读时间（如果需要，现在没装）：  

```
$ npm install hexo-symbols-count-time --save
```

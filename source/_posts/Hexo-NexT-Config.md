---
title: Hexo + NexT 博客配置备忘
date: 2019-03-27 18:59:38
tags: [Hexo, NexT]
categories: [备忘]
permalink: Hexo-NexT-Config
---
<center> <font color="#bababa">***个人 Blog 配置备忘***</font><br /> </center>
<!--more-->
---
# 基本信息  
建站驱动：`Hexo` v3.8.0  

主题： `NexT`.`Pisces` v7.0.1  

站点配置文件：`/_config.yml`  

主题配置文件：`/themes/next/_config.yml`  

# 我的配置  

## 选用主题  

`站点配置文件`：  

```yaml
theme: next     # 这项改成这个
```

## 文章按更新时间排序  

默认是按文章创建时间排序。要改成更新时间，去`站点配置文件`：  

```yaml
index_generator:
  path: ''
  per_page: 10
  order_by: -updated    # 这项改成这个
```

## 选用主题皮肤  

`主题配置文件`：  

```yaml
# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces      # 要哪个就取消注释哪个
#scheme: Gemini
```


## 隐藏页脚 Hexo 和 NexT 的声明和版本  

`主题配置文件`：  

```yaml
  copyright:
  # -------------------------------------------------------------
  powered:
    # Hexo link (Powered by Hexo).
    enable: false                        # 这项改成false
    # Version info of Hexo after Hexo link (vX.X.X).
    version: true

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: false                        # 这项改成false
    # Version info of NexT after scheme info (vX.X.X).
    version: true
  # -------------------------------------------------------------
```

## sidebar 放到右侧  

`主题配置文件`：  

```yaml
sidebar:
  # Sidebar Position, available values: left | right (only for Pisces | Gemini).
  #position: left
  position: right
```

## 增加返回顶部按钮、样式  

`主题配置文件`：

```yaml
  # Back to top in sidebar.
  b2t: false
  # Scroll percent label in b2t button.
  scrollpercent: true
  # Enable sidebar on narrow view (only for Muse | Mist).
  onmobile: false
  # Click any blank part of the page to close sidebar (only for Muse | Mist).
  dimmer: false
```

## 配置侧边栏菜单内容  

`主题配置文件`：

```yaml
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

目前启用主页、标签、分类、归档。`||`后面是`FontAwesome`样式的 icon 名字。

## 增加站内搜索  

在`hexo init`目录下安装插件：  

```
  npm install hexo-generator-search
  npm install hexo-generator-searchdb
```

`站点配置文件`中末尾增加内容：  

```yaml
# rtk: local search setting
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

`主题配置文件`中启用本地搜索：  

```yaml
# Local search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true      # 这项改成true来启用
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
```

## 主页侧边栏增加社交信息  

`主题配置文件`：  

```yaml
social:
  GitHub: https://github.com/xxx || github
  #E-Mail: mailto:yourname@gmail.com || envelope
  #Weibo: https://weibo.com/yourname || weibo
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #VK Group: https://vk.com/yourname || vk
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  Instagram: https://instagram.com/xxx || instagram
  #Skype: skype:yourname?call|chat || skype

  social_icons:
  enable: true
  icons_only: true    # 只显示图标
  transition: false
```

同样的，`||`后面是`FontAwesome`样式的 icon 名字。

## 增加 Valine 评论功能  

去 [LeanCloud](https://leancloud.cn/) 注册并创建一个免费应用，获得应用的`App ID`、`App Key`。在 `LeanCloud - 设置 - 安全中心 - Web 安全域名`中填上自己博客的地址。  

然后去`主题配置文件`内搜`Valine`，`enable`改成`true`，填上`App ID`和`App Key`：

```yaml
# Valine
# You can get your appid and appkey from https://leancloud.cn
# More info available at https://valine.js.org
valine:
  enable: true # When enable is set to be true, leancloud_visitors is recommended to be closed for the re-initialization problem within different leancloud adk version.
  appid: xxx # your leancloud application appid
  appkey: xxx # your leancloud application appkey
  notify: false # mail notifier, See: https://github.com/xCss/Valine/wiki
  verify: false # Verification code
  placeholder: ヾﾉ≧∀≦)o # comment box placeholder
  avatar: mm # gravatar style
  guest_info: nick,mail,link # custom comment header
  pageSize: 10 # pagination size
  visitor: false # leancloud-counter-security is not supported for now. When visitor is set to be true, appid and appkey are recommended to be the same as leancloud_visitors' for counter compatibility. Article reading statistic https://valine.js.org/visitor.html
  comment_count: true # if false, comment count will only be displayed in post page, not in home page
```

为了加快打开速度，去`/theme/next/layout/_third-party/comments/valine.swig`把：  
`//unpkg.com/valine/dist/Valine.min.js` 改成：  
`//cdn.jsdelivr.net/npm/valine/dist/Valine.min.js`  

[Valine 详细配置过程还可参见这里](https://www.jianshu.com/p/728a9594bb6c)

## 不显示文章信息中“评论数”三个字  

在`/themes/next/layout/_macro/post.swig`里，用注释符号把下面的第三行注释掉：  

```
{% elif (is_post() or theme.valine.comment_count) and theme.valine.enable and theme.valine.appid and theme.valine.appkey %}
    {{ comments() }}
         {#<span class="post-meta-item-text">{{ __('post.comments_count') + __('symbol.colon') }}</span>#}
        <a href="{{ url_for(post.path) }}#comments" itemprop="discussionUrl">
        <span class="post-comments-count valine-comment-count" data-xid="{{ url_for(post.path) }}" itemprop="commentCount"></span>
        </a>
    </span>
{% endif %}
```

## 增加页脚显示网站运营时间功能  

在`/themes/next/layout/_parrials/footer.swig`末尾增加内容：  

```javascript
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
    document.getElementById("sitetime").innerHTML=" 已运行"+/*diffYears+" 年 "+*/diffDays+" 天 "+diffHours+" 小时 "+diffMinutes+" 分钟 "+diffSeconds+" 秒 ~喵~";
  }/*因为建站时间还没有一年，就将之注释掉了。需要的可以取消*/
  siteTime();
</script>
```

## 增加页脚不蒜子的站点访问人次统计  

（接上小节代码）在`/themes/next/layout/_parrials/footer.swig`末尾增加内容：  

```javascript
</br>
<!--这一段是不蒜子的访问量统计代码-->
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<span class="post-meta-item-icon">
    <i class="fa fa-paw"></i>
</span>   // 猫爪图标
<span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次 &nbsp;   </span>
<span id="busuanzi_container_site_uv">访客数<span id="busuanzi_value_site_uv"></span>人次</span>
```

## 增加文章阅读次数显示  

在`/themes/next/layout/_macro/post.swig`的第一行中，增加一个`is_pv`字段：  

```javascript
{% macro render(post, is_index, is_pv, post_extra_class) %}
```

然后在`/themes/next/layout/_macro/post.swig`中增加一段代码：  

```javascript
          {% if is_pv %}
            <span class="post-meta-divider">|</span>
            <span class="post-meta-item-icon">
                <i class="fa fa-file-o"></i>
            </span>
            <span id="busuanzi_value_page_pv"></span>次阅读
          {% endif %}
```

上面代码塞的位置是：  

```javascript
          {% if not is_index and theme.busuanzi_count.enable and theme.busuanzi_count.post_views %}
            <span class="post-meta-divider">|</span>
            <span class="post-meta-item-icon"
            {% if not theme.post_meta.item_text %} title="{{ __('post.views') }}" {% endif %}>
            <i class="fa fa-{{ theme.busuanzi_count.post_views_icon }}"></i>
            {% if theme.post_meta.item_text %} {{ __('post.views') + __('symbol.colon') }} {% endif %}
            <span class="busuanzi-value" id="busuanzi_value_page_pv"></span>
            </span>
          {% endif %}

          {% if is_pv %}
            <span class="post-meta-divider">|</span>
            <span class="post-meta-item-icon">
                <i class="fa fa-file-o"></i>
            </span>
            <span id="busuanzi_value_page_pv"></span>次阅读
          {% endif %}

          {% if config.symbols_count_time.symbols or config.symbols_count_time.time %}
            <div class="post-symbolscount">
              {% if not theme.symbols_count_time.separated_meta %}
                <span class="post-meta-divider">|</span>
              {% endif %}
```

然后去`themes/next/layout/post.swig`（文章模板），给`render`方法放入参数`false`和`true`（分别对应`is_index`和`is_pv`：  

```javascript
{% block content %}

  <div id="posts" class="posts-expand">
    {{ post_template.render(page, false, true) }}
  </div>

{% endblock %}
```

再去`/themes/next/layout/index.swig`（首页模板），给`render`传入参数`true`和`false`。

```javascript
{% block content %}
  <section id="posts" class="posts-expand">
    {% for post in page.posts %}
      {{ post_template.render(post, true,false) }}
    {% endfor %}
  </section>
```

## 增加文章边框阴影、取消文章间分割线  

在`\themes\next\source\css_custom\custom.styl`文件中写入：  

```css
// 主页文章添加阴影效果
.post {
   margin-top: 0px;
   margin-bottom: 24px;
   padding: 35px;
   -webkit-box-shadow: 0 0 24px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```

增加了文章边框后，文章间的分割线就没必要存在了，所以干脆取消。  

在`/themes/next/source/css/_common/components/post/post-eof.styl`中把分割线高度改成`0px`,并把`margin`一行注释掉：  

```css
.posts-expand {
  .post-eof {
    display: block;
    // margin: $post-eof-margin-top auto $post-eof-margin-bottom;
    width: 0%;
    height: 0px;
    background: $grey-light;
    text-align: center;
  }
}
```

## 首页展示文章开头，增加`阅读全文`按钮  

`主题配置文件`中：

```yaml
# Automatically Excerpt. Not recommend.
# Use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: true          # 改成true
  length: 150

# Read more button
# If true, the read more button would be displayed in excerpt section.
read_more_btn: true     # 改成true
```

## 修改新建文章模板  

`/scaffolds/post.md`中修改内容：  

```go
---
title: {{ title }}
date: {{ date }}
tags:           // 文章标签
categories:     // 文章分类
permalink:      // 文章自定义链接
---
<center> <font color="#bababa">***首页展示的字***</font><br/> </center>
<!--more-->
---
```

## 修改文章末尾 tags 的符号（`#`改成图标）  

在`/themes/next/layout/_macro/post.swig中`，搜索`rel="tag">#`，将`#`换成`<i class="fa fa-tag"></i>`。  


## SEO 相关  

### 主题自带的 SEO 选项  

在`主题配置文件中`修改项：  

```yaml
seo: true
```


### 站点地图  

安装：  

```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

在`站点配置文件`末尾加入内容：  

```yaml
# rtk: sitemap setting
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

在`/source`中建一个`robots.txt`：

```
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/ 
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

sitemap: https://reitake.github.io/sitemap.xml
```

# 遗留问题  

## 文章边框  

加了文章边框后，首页看起来是不会干巴巴的了，但是在竖屏手机上看，可展示文章区域就窄了，体验不佳。（所以需要pad？

## 百度抓取  

目前 github 屏蔽了百度的抓取，所以就先没做`baidusitemap`和`百度站长`内容，以后不屏蔽了再说吧。

# 踩过的坑  

## `fontawesome` icon 显示不出来  

`主题配置文件中`，把这一项按 example 填上：  

```yaml
  # Internal version: 4.6.2
  # See: https://fontawesome.com
  # Example:
  # fontawesome: //cdn.jsdelivr.net/npm/font-awesome@4/css/font-awesome.min.css
  # fontawesome: //cdnjs.cloudflare.com/ajax/libs/font-awesome/4.6.2/css/font-awesome.min.css
  fontawesome: //cdn.jsdelivr.net/npm/font-awesome@4/css/font-awesome.min.css
```

给了两个 example，试着 ping 了下，第一个 8 ms，第二个 280 ms，选用了第一个。  

ps. 看了下其他人的 NexT v5.x 的配置文件，这一项默认是空着的，但能显示出 icon，现在用的 v7.0.1 也是默认空着，但我这只有填上才能显示出来，不知道是不是八阿哥。  


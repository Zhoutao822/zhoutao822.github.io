---
title: "Hugo Theme PaperMod"
date: 2021-12-12T20:55:38+08:00
tags: ["hugo", "papermod"]
categories: ["tools"]
series: ["hugo on mac"]
summary: "PaperMod configurations, favicon generator and mathjax issues"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

## 1. Favicon Generator

首先为网站添加一个Icon，这里直接使用这个网站[Favicon Generator. For real.](https://realfavicongenerator.net/)，上传任意一张喜欢的图片即可生成各种平台需要的favicon。点击下载，将所有图片资源保存在`static`目录下

![Screen Shot 2021-12-13 at 21.40.22](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112132246967.png)

![Screen Shot 2021-12-14 at 21.51.14](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112142151006.png)

## 2. PaperMod Theme Config

### 2.1 Favicon

在上一步保存了整个网站的Icon资源，接下来可以在`config.yaml`文件中配置favicon

```yaml
params:
  assets:
    # disableHLJS: true # to disable highlight.js
    # disableFingerprinting: true
    favicon: /favicon.ico
    favicon16x16: /favicon-16x16.png
    favicon32x32: /favicon-32x32.png
    apple_touch_icon: /apple-touch-icon.png
    safari_pinned_tab: /favicon.ico

  label:
    text: "Home"
    icon: /apple-touch-icon.png
    iconHeight: 35
```

### 2.2 Taxonomies

Taxonomies用于给所有博客按标签归档，默认支持三种`categories`、`tags`、`series`，需要在`config.yaml`中声明：

```yaml
taxonomies:
    category: categories
    tag: tags
    series: series
```

只需要在markdown中声明即可自动归档

```
---
title: "Hugo On Mac"
date: 2021-12-12T11:18:58+08:00
tags: ["hugo", "github pages", "typora", "picgo", "mathjax", "utteranc"]
categories: ["tools"]
series: ["hugo on mac"]
summary: "Hugo + Github Pages + Typora搭建Markdown博客"
draft: false
---
```

开启后可以在将菜单选项展示到页面顶部，weight决定菜单顺序：

```yaml
menu:
  main:
    - name: Archive
      url: archives/
      weight: 5
    - name: Tags
      url: tags/
      weight: 10
    - name: Categories
      url: categories/
      weight: 15
    - name: Series
      url: series/
      weight: 20
    - name: Search
      url: search/
      weight: 25
```

### 2.3 Search && Archive

Search也是PaperMod主题支持的，首先添加文件`content/search.md`

```
---
title: "Search" # in any language you want
layout: "search" # is necessary
url: "/search/"
# description: "Description for Search"
summary: "search"
---
```

然后修改`config.yaml`

```yaml
outputs:
    home:
        - HTML
        - RSS
        - JSON # is necessary
```

还有一些搜索相关参数保持默认即可

```yaml
params:
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]
```

Archive的支持只需要添加文件`content/archives.md`

```
---
title: "Archive"
layout: "archives"
url: "/archives/"
summary: archives
---
```

最终效果如下：

![Screen Shot 2021-12-15 at 21.15.51](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112152116458.png)

![Screen Shot 2021-12-15 at 21.16.05](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112152116435.png)

![Screen Shot 2021-12-15 at 21.16.11](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112152116193.png)

### 2.4 Archetypes

Hugo支持默认模板，当执行`hugo new blog.md`时使用模板生成`blog.md`，可以在创建markdown时自动添加部分属性，修改`archetypes/default.md`

```
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
tags: [""]
categories: [""]
series: [""]
summary: "Summary todo"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---
```

### 2.5 其他设置

```yaml
baseURL: 'https://zhoutao822.github.io/'
languageCode: en-us
title: Tao's Notes
theme: PaperMod
paginate: 5

enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false

googleAnalytics: UA-123-45

minify:
  disableXML: true
  minifyOutput: true

params:
  env: production # to enable google analytics, opengraph, twitter-cards and schema.
  title: Tao's Notes
  description: "Tao's learning notes"
  keywords: [Blog, Portfolio, PaperMod]
  author: Me
  images: ["<link or path of image for opengraph, twitter-cards>"]
  DateFormat: "January 2, 2006"
  defaultTheme: dark # dark, light, auto
  disableThemeToggle: true

  ShowReadingTime: true
  ShowShareButtons: false
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  hidemeta: false
  hideSummary: false
  showtoc: true
  tocopen: false

  comments: true
  utteranc:
    enable: true
    repo: "zhoutao822/zhoutao822.github.io"
    issueTerm: "title"
    theme: "github-dark"

  assets:
    # disableHLJS: true # to disable highlight.js
    # disableFingerprinting: true
    favicon: /favicon.ico
    favicon16x16: /favicon-16x16.png
    favicon32x32: /favicon-32x32.png
    apple_touch_icon: /apple-touch-icon.png
    safari_pinned_tab: /favicon.ico

  label:
    text: "Home"
    icon: /apple-touch-icon.png
    iconHeight: 35

  # profile-mode
  profileMode:
    enabled: false # needs to be explicitly set
    title: ExampleSite
    subtitle: "This is subtitle"
    imageUrl: "<img location>"
    imageWidth: 120
    imageHeight: 120
    imageTitle: my image
    buttons:
      - name: Posts
        url: posts
      - name: Tags
        url: tags

  # home-info mode
  homeInfoParams:
    Title: "Hi there \U0001F44B"
    Content: Welcome to my blog

  socialIcons:
    - name: github
      url: "https://github.com/Zhoutao822/"

  analytics:
    google:
      SiteVerificationTag: "Tao"
    bing:
      SiteVerificationTag: "Tao"
    yandex:
      SiteVerificationTag: "Tao"

  cover:
    hidden: true # hide everywhere but not in structured data
    hiddenInList: true # hide on list pages and home
    hiddenInSingle: true # hide on single page

  editPost:
    URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]

outputs:
    home:
        - HTML
        - RSS
        - JSON

taxonomies:
    category: categories
    tag: tags
    series: series

menu:
  main:
    - name: Archive
      url: archives/
      weight: 5
    - name: Tags
      url: tags/
      weight: 10
    - name: Categories
      url: categories/
      weight: 15
    - name: Series
      url: series/
      weight: 20
    - name: Search
      url: search/
      weight: 25
```

## 3. Mathjax Defects

### 3.1 mathjax.html

首先需要修改`layouts/partials/mathjax.html`

```html
{{ if .Params.mathjax }}
<script>
    MathJax = {
        tex: {
            inlineMath: [["$", "$"]],
            processEscapes: true,
            processEnvironments: true
        },
        displayMath: [
            ["$$", "$$"],
            ["\[", "\]"],
        ],
        svg: {
            fontCache: "global",
        },
    };
</script>
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
{{ end }}
```

需要添加`processEscapes: true`，否则行内数学公式无法正常显示。

### 3.2 issues

```markdown
属性$a_k = \underset{a \in A}{\arg \max Gain(D, a)}$。

$$
Gain\_ratio(D, a) = \frac{Gain(D,a)}{IV(a)}
\\
IV(a) = -\sum^V_{v=1}\frac{|D^v|}{|D|} \log_2\frac{|D^v|}{|D|}
$$

$$
Gini(D) = \sum^{|\mathbb{Y}|}_{k=1}\sum_{k' \neq k}p_kp_{k'}
\\
= 1- \sum^{|\mathbb{Y}|}_{k=1}p_k^2
$$
```

![Screen Shot 2021-12-15 at 21.34.07](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112152134819.png)

就算修改了`mathjax.html`或者使用`katex`，这里无法避免转义字符的问题，期望的显示效果如上，但是实际效果如下

![Screen Shot 2021-12-15 at 21.34.54](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/202112152135966.png)

第一点就是数学公式中添加`\\`应该实现换行，这里换行失效，第二点就是`\_`转义字符期望显示`_`，但是变成下角标（`Gain\_ratio`），第三点就是某些公式可以在Typora中正常渲染，但是在Hugo中无法解析，推测也是跟Hugo渲染相关的问题，这个问题在Hexo中也有（[结合MathType和MathJax在Hexo博客中插入数学公式](https://zhuanlan.zhihu.com/p/108766968)），但是Hexo可以修改其源码来解决这个问题，Hugo不适合修改源码解决。所以最终结论是，在数学公式中**不要使用下划线，因为会解析错误**，**不要使用\\\\实现公式内换行**，**复杂公式需要检查**，当然也可以选择不要在Hugo中使用复杂公式。

## 4. PicGo图床

SM.MS图床有资源限制，建议使用gitee或者github作为图床，gitee存在单张图片不能超过2MB的限制，这里就需要使用PicGo的Compress插件[picgo-plugin-compress](https://github.com/JuZiSang/picgo-plugin-compress)并且使用lubangitee算法，在mac上安装这个插件可能会安装不上，建议~~翻墙~~或者通过命令行安装


```shell
# 首先要安装这些库，否则在mac上使用lubangitee压缩算法时会失败
brew install node automake autoconf libtool pkgconfig libpng nasm
```

然后进入到picgo目录中通过npm安装[picgo-plugin-compress](https://github.com/JuZiSang/picgo-plugin-compress)插件


```shell
cd ~/Library/Application\ Support/picgo
rm -rf node_modules
npm install picgo-plugin-compress --save --registry=https://registry.npm.taobao.org  --ignore-scripts
npm install --registry=https://registry.npm.taobao.org
```

如果在使用lubangitee出现以下错误时，说明上面几个库有可能漏掉了，解决方案来自[mozjpeg pre-build test failed](https://github.com/imagemin/imagemin/issues/168)


```
2022-01-06 20:18:41 [PicGo ERROR] 
------Error Stack Begin------
Error: spawn /Users/tao/Library/Application Support/picgo/node_modules/mozjpeg/vendor/cjpeg ENOENT
    at Process.ChildProcess._handle.onexit (internal/child_process.js:264:19)
    at onErrorNT (internal/child_process.js:456:16)
    at processTicksAndRejections (internal/process/task_queues.js:84:9)
-------Error Stack End------- 
```

## 参考

1. [Favicon Generator. For real.](https://realfavicongenerator.net/)
1. [GitHub + jsDelivr + PicGo + Imagine 打造稳定快速、高效免费图床](https://juejin.cn/post/6844903993529860109)
1. [结合MathType和MathJax在Hexo博客中插入数学公式](https://zhuanlan.zhihu.com/p/108766968)
1. [picgo-plugin-compress](https://github.com/JuZiSang/picgo-plugin-compress)

# JEKYLL OMEGA THEME

HMFAYSAL OMEGA 是一个最小化，很漂亮并且有着响应式设计的Jekyll主题，它为那些喜欢文章内容放在网页最上方和中间的工程师和文字工作者而设计。这个主题可以给每个页面和文章添加一个漂亮的背景图片。 它由工程师 slash Mathematician 创建[Hossain Mohd Faysal](http://alum.mit.edu/www/hmfaysal/).

主题特性:

* Twitter Bootstrap 3
* 可选择设置背景图片
* 自定义Js代码去强调你文章中的第一段 `p:first-child`
* 通过设置`post types`有多种模板，比如video,photo,status update
* 通过原生MathJax支持来显示公式
* 主题使用插件，因此可以直接push主题到github pages来使用
* 阅读时间计算器: 基于文章字数去计算大概的阅读时间 (通过Liquid Tags来实现而不是使用插件)
* 支持手动开关分页器
* 使用CSS3过渡的效果获得更好的性能
* 可以在_config.yml里设置动画
* 可读的typography排版让你的文字亮瞎眼
* 你可以选择打开Disqus评论或者多说评论
* 简单明了的永久链接
* 页脚菜单
* SVG图形
* Google Fonts
* 479个Fontawesome图标字体
* 支持 [Open Graph](https://developers.facebook.com/docs/opengraph/) 和 [Twitter Cards](https://dev.twitter.com/docs/cards) 带给你更好的社交分享体验
* 漂亮的 [404 页面](http://hmfaysal.github.io/hmfaysal-omega-theme/404.html)
* 自定义 [categories](http://hmfaysal.github.io/hmfaysal-omega-theme/categories/) 和 [tags](http://hmfaysal.github.io/hmfaysal-omega-theme/tags/) 页面
* 使用Solarized-light代码高亮样式
* 基于文章标题的Simple search
* 给搜索引擎使用的Sitemap
* 由一位电子工程师设计

![screenshot of HMFAYSAL OMEGA Theme](https://raw.github.com/hmfaysal/hmfaysal-omega-theme/gh-pages/images/hmfaysal-omega-preview.jpg)

## 最快部署

如果你不需要本地预览，完全不需要安装一堆的软件，只需要这样做：

1. github帐号和git
2. 一个支持markdown的编辑器
3. fork这个项目
4. clone之后本地提交修改

---

## 从已有的Jekyll站点安装

1. Clone下面的目录: `_includes`, `_layouts`, `plugins`, `assets`, and `images`.
2. Clone 下面列出的文件: `about.md`, `index.html`, `categories.html`, `tags.html`, `feed.xml`, 和 `sitemap.xml`.
3. 在你自己的 `_config.yml` 文件里设置下面的变量:

``` yaml
title:            Site Title
description:      Site description for the metas.
logo:             site-logo.png
disqus_shortname: shortname
duoshuo_shortname: 
# Assign a default image for your site's header and footer
default_bg:       some-image.jpg
search:           true
share:            true
# Read Time is a calculator tp provide post read-time based on word count. Usage is recommended.
readtime:         true
# Turn on or off the fin animations in the header and footer
animated_fins:    true
# Specify the fin color in RGB value
fin_color:        "255,255,255"
# Change url to your domain. Leave localhost server or blank when working locally.
url:              "http://localhost:4000"


# Owner/author information
owner:
  name:           Your Name
  avatar:         your-photo.jpg
  email:          your@email.com
  # Use the coder's toolbox at http://coderstoolbox.net/string/#!encoding=xml&action=encode&charset=us_ascii to encode your description into XML string
  description:	  Some Details about yourself
  # Social networking links used in footer. Update and remove as you like.
  # To register at HMFAYSAL SOCIAL, visit http://social.hmfaysal.tk
  twitter:
  facebook:
  github:
  linkedin:
  instagram:
  tumblr:
  hmfaysalsocial:
  # For Google Authorship https://plus.google.com/authorship
  google_plus:    "http://plus.google.com/123123123123132123"

# Analytics and webmaster tools stuff goes here
google_analytics:
google_verify:
# https://ssl.bing.com/webmaster/configure/verify/ownership Option 2 content= goes here
bing_verify:

# nprogress
nprogress:  true

# Links to include in top navigation
# For external links add external: true
links:
  - title: Home
    url: /
    external: false
    icon: home
  - title: Categories
    url: /categories
  - title: Tags
    url: /tags
  - title: Guestbook
    url: /guestbook
  - title: About
    url: /about

# http://en.wikipedia.org/wiki/List_of_tz_database_time_zones
timezone:    Asia/Shanghai
future:      true
highlighter: pygments
markdown:    kramdown
excerpt_separator: "<!--more-->"
paginate:    6
paginate_path: "page:num"

# https://github.com/mojombo/jekyll/wiki/Permalinks
permalink:   /:categories/:title

kramdown:
  auto_ids: true
  footnote_nr: 1
  entity_output: as_char
  toc_levels: 1..6
  use_coderay: false

  coderay:
    coderay_line_numbers: 
    coderay_line_numbers_start: 1
    coderay_tab_width: 4
    coderay_bold_every: 10
    coderay_css: class
```

---

## 文章头信息 YAML

HMFAYSAL OMEGA主题可以设置变量来使用相应的模板，包括 articles, quotation, video, photo和status updates.

一般文章开头要有下面的结构才能用到本主题的功能

``` yaml
---
layout: post
title: "Some Title"					# Title of the post
description: Some description		# Description of the post, used for Facebook Opengraph & Twitter
headline: Some headline				# Will appear in bold letters on top of the post
modified: YYYY-MM-DD				# Date
category: personal
tags: []
image: 
  feature: some-image.jpg
comments: true
mathjax:
---
```

"状态"类型文章的开头要有下面的结构

``` yaml
---
layout: post
type: status                # ! Important
title: "Some Title"         # Title of the post
description: Some description   # Description of the post, used for Facebook Opengraph & Twitter
headline: Some headline       # Will appear in bold letters on top of the post
modified: YYYY-MM-DD        # Date
category: personal
tags: []
image: 
  feature: some-image.jpg
comments: true
mathjax:
---
```

引用类型的文章使用下面的结构

``` yaml
---
layout: post
type:  quote                # ! Important
title: "Some Title"         # Title of the post
description: Some description   # Description of the post, used for Facebook Opengraph & Twitter
headline: Some headline       # Will appear in bold letters on top of the post
modified: YYYY-MM-DD        # Date
category: personal
tags: []
image: 
  feature: some-image.jpg
comments: true
mathjax:
---
```

视频类型文章的头信息结构

``` yaml
---
layout: post
type:  video                # ! Important
title: "Some Title"         # Title of the post
description: Some description   # Description of the post, used for Facebook Opengraph & Twitter
headline: Some headline       # Will appear in bold letters on top of the post
modified: YYYY-MM-DD        # Date
category: personal
tags: []
image: 
  feature: some-image.jpg
comments: true
mathjax:
---
```

图片类型文章的头信息结构

``` yaml
---
layout: post
type:  photo                # ! Important
photo: some-image.jpg 		# In case you do not want the featured image to display on the front page
title: "Some Title"         # Title of the post
description: Some description   # Description of the post, used for Facebook Opengraph & Twitter
headline: Some headline       # Will appear in bold letters on top of the post
modified: YYYY-MM-DD        # Date
category: personal
tags: []
image: 
  feature: some-image2.jpg
comments: true
mathjax:
---
```

---

## 目录结构
``` bash
HMFAYSAL-OMEGA-THEME
│
│
├───assets
│   ├───css
│   │       bootstrap.css
│   │       style.css
│   │
│   ├───fonts
│   │   ├───glyphicons─halflings─regular.eot
│   │   │       index.html
│   │   │
│   │   ├───glyphicons─halflings─regular.svg
│   │   │       index.html
│   │   │
│   │   ├───glyphicons─halflings─regular.ttf
│   │   │       index.html
│   │   │
│   │   └───glyphicons─halflings─regular.woff
│   │           index.html
│   │
│   └───js
│       │   script.js
│       │   scripts.min.js
│       │   waypoints.min.js
│       │   _main.js
│       │
│       ├───plugins
│       │       jquery.fitvids.js
│       │       jquery.magnific─popup.js
│       │       simpleJekyllSearch.js
│       │
│       └───vendor
│               jquery─1.9.1.min.js
│
├───images
│
├───_includes
│       browser─upgrade.html
│       disqus_comments.html
│       footer.html
│       head.html
│       header.html
│       scripts.html
│       signoff.html
│
├───_layouts
│       home.html
│       page.html
│       post.html
│
└───_posts
```

## 许可证

This theme is free and open source software, distributed under the [The MIT License](LICENSE). So feel free to use this Jekyll theme on your site without linking back to me or using a disclaimer.

If you'd like to give me credit somewhere on your blog or tweet a shout out to [@hmfaysal](https://twitter.com/hmfaysal), that would be pretty sweet.


Warm Regards and Stay Creative,
Hossain Mohd. Faysal

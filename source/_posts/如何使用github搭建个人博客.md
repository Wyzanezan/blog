---
title: 如何使用github搭建个人博客
date: 2019-11-28 19:52:40
tags:
---

最近使用 hexo + github 搭建了一个博客系统，现总结搭建步骤并记录如下。

<!--more-->

## 1. 安装相关软件

在windows上安装 node.js，npm，git，具体安装步骤可以百度。



## 2. 创建github仓库

在github上创建名称为 你的用户名.github.io 的仓库，后面搭建成功后，可以在浏览器上输入 https://你的用户名.github.io 来访问博客。

仓库创建成功后，需要配置SSH Key，以便后续提交博客到仓库中。在windows下 git 的命令行中输入 ssh-keygen 生成 公钥和私钥（公、私钥的文件名称使用默认的即可，不使用默认的文件名后需提交博客可能提示无权限），生成文件后，将 id_rsa.pub 中的内容添加到 仓库 --> Settings --> Deploy key 中。



## 3. 使用hexo写博客

hexo是一个快速、简单、功能强大的博客框架，在Markdown中写好博客后，hexo可以使用优美的主题将博客生成静态文件。 其官网如下：[http://hexo.io](http://hexo.io/)

### 3.1 安装hexo并初始化

使用 npm 安装 hexo：

```js
npm install -g hexo
```

然后在本地文件夹新建 hexo工作目录并初始化该目录：

```js
cd D:\hexo
hexo init
```

初始化过程中会下载一些 js 模块，存放在 node_modules 文件夹下。

然后可以执行下面的命令：

```js
hexo g  # hexo generate 的缩写，生成静态页面至public目录下
hexo s  # hexo server 的缩写， 启动服务 （默认 http://localhost:4000）
```

在浏览器上访问 http://localhost:4000，可以看到有一篇默认的博客和主题。



### 3.2 更换主题

官方主题：[https://hexo.io/themes/](https://hexo.io/themes/)

可以到这里寻找自己喜欢的主题，并将其克隆到 hexo工作目录的themes目录下；然后在hexo目录下的_config.yml文件中修改 theme的配置（例如 修改 theme: landscape 为 theme: clean-blog）；最后执行hexo g命令即可。

更改完主题后，可以重启服务，访问 http://localhost:4000，查看主题的效果。



### 3.3 deploy配置

在 _config.yml 文件中修改下面的配置：

```txt
deploy:
  type: git
  repository: git@github.com:wyzane/wyzane.github.io.git
  branch: master
```

将 repository 的值修改成你自己的github仓库名，修改完成后，执行 hexo d（hexo deploy）就可以将代码提交至执行的github仓库中。



### 3.4 编写博客

执行 hexo new "文章名称" 会在 hexo的 source\\_posts目录下新建一篇文章，文章是Markdown格式，推荐使用Typora软件来编辑md文件。

编辑好文章后，执行 hexo g命令，会将md文件转换成html静态文件并添加到public目录下对应的目录中，最后执行 hexo d将更新提交至github仓库中。

访问上面提到的地址，就可以看到新增的博客文章。



### 3.5 hexo常用命令

```js
hexo new "name" # 新建文章
hexo new page "name" # 新建页面
hexo generate # 生成静态页面至public目录
hexo server # 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy # 部署到GitHub
hexo help  # 查看帮助
hexo version  # 查看Hexo的版本

# 缩写如下：
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```



### 3.6 _config.yml文件

_config.yml文件是 hexo 的配置文件，里面主要有以下配置：

```yml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: wyzane's skill blog
subtitle: ''
description: ''
keywords:
author: wyzane
language: en
timezone: ''

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://wyzane.cc
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing index.html from permalinks

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''

# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Metadata elements
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
## Use post's date for updated date unless set in front-matter
use_date_for_updated: false

# Pagination
## Set per_page to 0 to disable pagination
per_page: 6
pagination_dir: page

# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include:
exclude:
ignore:

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: clean-blog

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:wyzane/wyzane.github.io.git
  branch: master
```

_config.yml 文件中可以配置标题、主题、分页等参数。



使用 hexo + github 搭建博客有以下优点：

1. 简单，成本低，
2. 支持使用Markdown编写博客，
3. 扩展对 node.js，hexo，github等知识的应用。



更多详细信息可以参考这篇文章：[http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa](http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa)










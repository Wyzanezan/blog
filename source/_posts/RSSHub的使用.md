---
title: RSSHub的使用
date: 2020-04-08 23:00:23
tags: 
categories: Nodejs
---

RSS简易信息聚合（也叫[聚合内容](https://www.baidu.com/s?wd=聚合内容&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)）是一种RSS基于XML标准，在互联网上被广泛采用的内容包装和投递协议。

<!--more-->

# RSS简介

​		RSS简易信息聚合（也叫[聚合内容](https://www.baidu.com/s?wd=聚合内容&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)）是一种RSS基于XML标准，在互联网上被广泛采用的内容包装和投递协议。RSS(Really Simple Syndication)是一种描述和同步网站内容的格式，是使用最广泛的[XML应用](https://www.baidu.com/s?wd=XML应用&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。RSS搭建了信息迅速传播的一个技术平台，使得每个人都成为潜在的信息提供者。发布一个RSS文件后，这个RSS Feed中包含的信息就能直接被其他站点调用，而且由于这些数据都是标准的[XML格式](https://www.baidu.com/s?wd=XML格式&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)，所以也能在其他的终端和服务中使用，是一种描述和同步网站内容的格式。[1] RSS可以是以下三个解释的其中一个： Really Simple Syndication；RDF (Resource Description Framework) Site Summary； Rich Site Summary。但其实这三个解释都是指同一种Syndication的技术。
​		RSS目前广泛用于网上[新闻频道](https://www.baidu.com/s?wd=新闻频道&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)，blog和wiki，主要的版本有0.91, 1.0, 2.0。使用[RSS订阅](https://www.baidu.com/s?wd=RSS订阅&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)能更快地获取信息，网站提供RSS输出，有利于让用户获取网站内容的最新更新。网络用户可以在客户端借助于支持RSS的聚合工具软件，在不打开网站内容页面的情况下阅读支持RSS输出的网站内容。

​		其实，RSS的功能类似于爬虫，只是这个“爬虫”已经实现好了，我们只需要遵循它的规则去调用就可以获取所需的数据。

# RSSHub服务搭建

## RSSHub介绍

​		RSSHub 是一款轻量、易于扩展的 RSS 生成器，基于 node.js 10，可以给「任何」内容生成 RSS 订阅源，目前支持微信公众号、B 站、微博、即刻、网易云音乐、掘金、简书、知乎、自如、快递、贴吧、豆瓣、pixiv 等内容。

​		RSSHub 不是阅读器，它是一个用来生成 RSS 源的工具，并且你需要一台能够运行 node.js 的电脑来长期运行 RSSHub，当然推荐使用 VPS，除了能够实现稳定的源以外，还能抓取 Instagram 等内容源。

​		与 Huginn、Feed43 等工具类似，RSSHub 在大部分网站上也是通过抓取网页的方式获得订阅源，不同的是在 RSSHub 中，已经完成了对抓取规则的编写，只需要用户简单的编辑下地址即可。





## 搭建RSS准备工作

### 安装nodejs和npm

```txt
安装nodejs和npm就略过了
```



### npm镜像源

```txt
查看镜像源：
npm config get registry（默认结果是：http://registry.npmjs.org）

更改默认镜像源：
npm config set registry https://registry.npm.taobao.org
npm config set registry http://r.cnpmjs.org/

执行npm install时指定镜像源：
npm install --registry=https://registry.npm.taobao.org

国内其他镜像源：
淘宝npm镜像
搜索地址：http://npm.taobao.org/
registry地址：http://registry.npm.taobao.org/
cnpmjs镜像
搜索地址：http://cnpmjs.org/
registry地址：http://r.cnpmjs.org/
```



### node升级

```txt
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
```



## 搭建RSS服务步骤

### 搭建步骤

```txt
1. git clone https://github.com/DIYgod/RSSHub.git
2. cd RSSHub 并执行 npm install（执行速度慢时可以考虑更换npm源）
3. 启动rss服务：npm start
4. 在浏览器中输入 http://ip:port 即可查看
```



### 相关文章

```txt
https://www.moerats.com/archives/587/
https://www.jianshu.com/p/e17258dd5b69
https://sspai.com/post/47100
```



# RSSHub的使用

```txt
文档：https://docs.rsshub.app/
```



## 微博

```txt
根据用户id获取用户动态：
http://localhost:1200/weibo/user/:uid
例子：http://localhost:1200/weibo/user/2803301701

根据关键词获取用户动态：
http://localhost:1200/weibo/keyword/:keyword

获取热搜榜：
http://localhost:1200/weibo/search/hot
```



## 简书

```txt
获取首页数据：
http://localhost:1200/jianshu/home

根据用户id获取用户文章：
http://localhost:1200/jianshu/user/:uid
例子：http://localhost:1200/jianshu/user/c5a2ce84f60b

根据专题id获取文章：
http://localhost:1200/jianshu/collection/:cid
例子：http://localhost:1200/jianshu/collection/7b2be866f564
```



## 知乎

```txt
根据用户id获取知乎动态：
http://localhost:1200/zhihu/people/activities/:uid
例子：http://localhost:1200/zhihu/people/activities/yuansu-27-1

根据用户id获取用户回答的问题：
http://localhost:1200/zhihu/people/answers/:uid
例子：http://localhost:1200/zhihu/people/answers/yuansu-27-1

根据用户id获取用户的文章：
http://localhost:1200/zhihu/people/posts/:uid
例子：http://localhost:1200/zhihu/people/posts/yuansu-27-1

获取知乎热榜：
http://localhost:1200/zhihu/hotlist

知乎想法热榜：
http://localhost:1200/zhihu/pin/hotlist

根据问题id查找答案：
http://localhost:1200/zhihu/question/:qid
例子：http://localhost:1200/zhihu/question/352335277

根据专题id获取专题信息：
http://localhost:1200/zhihu/topic/:tid
例子：http://localhost:1200/zhihu/topic/21276827
```



## 今日头条

```txt
根据关键字查询信息：
http://localhost:1200/jinritoutiao/keyword/:keyword
```



## 腾讯NBA

```txt
http://localhost:1200/nba/app_news
```



## 雪球

```txt
根据用户id和类型获取用户动态：
http://localhost:1200/xueqiu/user/:uid/:type
例子：http://localhost:1200/xueqiu/user/8152922548/2
0-原发布
2-长文
4-问答
9-热门
11-交易

获取用户的收藏：
http://localhost:1200/xueqiu/favorite/:uid
例子：http://localhost:1200/xueqiu/favorite/8152922548

查询股票信息：
http://localhost:1200/xueqiu/stock_info/:id/:type
id：股票代码
type：动态类型
	announcement-公告
	news-新闻
	research-研报
例子：http://localhost:1200/xueqiu/stock_info/SZ399001
```



rsshub支持的其他网站可以查看下面的文档：

```txt
https://docs.rsshub.app/
```



# 其他可用RSS推荐

```txt
https://www.cnblogs.com/buwuliao/p/8379549.html（https://rssfeed.today/weibo/rss/:userid）
https://rssfeed.today/weibo/rss/:userid
```


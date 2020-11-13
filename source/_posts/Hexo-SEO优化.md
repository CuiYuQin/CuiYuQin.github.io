---
title: hexo-seo优化技巧
date: 2020-06-16 22:32:38
id: hexo-seo
tags:
    - 技术
categories: 技术
---


搭建的hexo博客如果不经过优化，在搜索引擎被收录的可能很小，SEO可以提高被搜到的几率


<a name="04ad26dd"></a>
### 一. 生成 sitemap 文件

需要先安装两个 hexo 插件：

```shell
npm install hexo-generator-sitemap --save		
npm install hexo-generator-baidu-sitemap --save
```

打开配置文件`_config.yml`添加

```yaml
sitemap:
	path: sitemap.xml
baidusitemap:
	path: baidusitemap.xml
```

再重启 hexo，在本地访问 [localhost:4000/sitemap.xml]()和 [localhost:4000/baidusitemap.xml]() 就能正确的展示出两个sitemap 文件了。

<!-- more -->

<a name="0951f961"></a>
### 二. 推送到 谷歌 和 百度

<a name="c58b62e5"></a>
#### 1. 百度 → [添加个人网站](https://ziyuan.baidu.com/site/siteadd?siteurl=)

添加文件方式不可行，hexo会处理html文件

所以选择，在 head.ejs 里添加 html 标签

1.1 [手动提交baidusitemap.xml](https://ziyuan.baidu.com/linksubmit/index)(里面也有自动提交的代码)

1.2 可以用"抓取诊断"，手动-百度抓取

1.3 Robots → 检测并更新

<a name="53dfa5ae"></a>
#### 2. 谷歌 → [添加个人网站](https://search.google.com/search-console/welcome)

类似百度 ，也是在 head.ejs 里添加 html 标签

> 验证通过就好，过两天左右 百度和谷歌就能收录你的站点

> 测试方式: (分别在 google 和 baidu 搜索)

> site: shen-yu.gitee.io


<a name="dc54d512"></a>
##### 2.1 手动提交sitemap，甚至是单个网站

<a name="19cedad5"></a>
##### [GoogleSearchConsole](https://search.google.com/search-console) → 站点地图 → 输入sitemap.xml → 提交

<a name="39dffe8d"></a>
##### 2.2 robots配置

```
User-agent: *
Allow: /
Allow: /home/
Allow: /archives/
Allow: /about/
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: http://yoursite/sitemap.xml
Sitemap: http://yoursite/baidusitemap.xml
```

Allow表示允许被访问的，Disallow是不允许的意思。注意后面两个Sitemap就是网站地图了。而网站地图前面说了是给爬虫用的。这里配置在robots中。

<a name="c35bfb76"></a>
##### 2.3 测试

旧版 GoogleSearchConsole 测试 robots.txt  是否配置好

新版 GoogleSearchConsole 测试 sitemap.xml 是否配置好

<a name="3676c080"></a>
### 三. 定期清除死链接

[https://www.google.com/webmasters/tools/removals](https://www.google.com/webmasters/tools/removals)
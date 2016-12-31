---
layout: post
title: "Jekyll博客搭建"
date: 2015-01-23
comments: true
categories: iOS
tags: [Jekyll]
keywords: Jekyll blog 迁移到gitcafe
description: Jekyll博客搭建, 迁移到gitcafe
---

###注意：###


由于博客已经换了搭建平台和主题，之前文章的评论已丢失，在这里对那些做过评论的网友说声抱歉。

把博客平台换成[Jekyll](http://jekyllcn.com/)，主要原因是这款令我满意的主题，而且Jekyll搭建和操作也非常简单。下面介绍在Mac OS X环境下如何通过Jekyll搭建自己的个人博客。

#### 搭建环境

首先安装必要工具

* [Ruby](https://www.ruby-lang.org/en/downloads/)：Mac OS X 10.5以上都自带
* [RubyGems](https://rubygems.org/pages/download)：Mac OS X 10.5以上都自带
* [NodeJS](http://nodejs.org/)：命令行输入`node -v`检查是否已安装，下载地址：http://nodejs.org/
* Xcode Command-Line Tools： 安装Xcode会自动安装，检查`Preferences → Downloads → Components`是否有Command-Line Tools这项提供下载，如果没有说明已安装
* [git](http://sourceforge.net/projects/git-osx-installer/)：命令行输入`git --version`检查是否已安装，下载地址：http://sourceforge.net/projects/git-osx-installer/

由于国内网络问题，必须要替换下gem镜像源，除非你有其他安全上网的方法

``` objectivec
// 移除官方镜像源
gem sources --remove https://rubygems.org/
// 添加淘宝镜像源，或者其他镜像地址
gem sources -a http://ruby.taobao.org/

```

验证是否替换成功`gem sources -l`

``` ruby
*** CURRENT SOURCES ***
http://ruby.taobao.org/
```

然后先更新下 `gem`

``` ruby
sudo gem update --system
// MAC 系统版本如果是 El Capitan 使用下面这个命令
sudo gem update -n /usr/local/bin --system
```

安装Jekyll

``` objectivec
sudo gem install jekyll
// MAC 系统版本如果是 El Capitan 使用下面这个命令
sudo gem install -n /usr/local/bin jekyll
```

如果你在后续预览 `jekyll server` 博客的发生

``` objectivec
Deprecation: Collection#map should be called on the #docs array directly.
                    Called by /Users//Documents/blog/_plugins/rssgenerator.rb:46:in `block in generate'.
```

类似的错误，请安装低于 3.0版本的 jeykll

``` objectivec
sudo gem install jekyll -v '<3.0.0'
// MAC 系统版本如果是 El Capitan 使用下面这个命令
sudo gem install -n /usr/local/bin jekyll -v '<3.0.0'
```

#### 安装主题

1. Fork我使用的这款主题[kasper](https://github.com/rosario/kasper)

2. 把fork后的项目名改为：`xxxxxx.github.io`xxxx为你的github用户名，比如我的用户名是bawn，那么就需要修改成`bawn.github.io`，这个也正是你博客的地址。

   点击项目右侧 settings 菜单，进入后修改 `Repository name`就是了。

完成这两步之后，在你的浏览器上输入xxxxxx.github.io或者xxxxxx.github.com，就会出现你个人博客的页面(可能需要等待几分钟)，这时候你的博客上	应该有一篇主题作者的默认文章叫做Welcome to Jekyll!



#### 发表文章

``` objectivec
// 进入主题需要放置的目录
cd Documents
// 克隆刚才fork的主题
git clone https://github.com/xxxx/xxxxx.github.io
```

完成之后你的Documents目录下应该有个xxxxx.github.io的文件夹（称之为博客目录），`_posts`文件夹中的markdown文件就是所发表的文章的源文件。

新建一篇文章的命名规则是`xxxx-xxx-xx-xxxxxxx.md`，比如我这篇文章的命名是`2015-01-23-bolgSetUp`，里面内容也需要一定的规则，下面是这篇文章的头部的写法

``` objectivec

---
layout: post
title: "Jekyll博客搭建"
date: 2015-01-23
comments: true
categories: iOS
tags: [Jekyll]
keywords: Jekyll blog
description: Jekyll博客搭建
---

由于博客已经换了搭建平台和主题，之前文章的评论已丢失，在这里对那些做过评论的网友说声抱歉。
```

Jekyll 官方文档比较坑，推荐的是用[Liquid](http://liquidmarkup.org/)方式的markdown解析语法来写代码块，比如是这样的：![image](/images/Jekyll/code.png)

<s>其实Jekyll还支持[redcarpet]( https://github.com/vmg/redcarpet)解析器，所以使用 markdown [GFM](https://help.github.com/articles/github-flavored-markdown/) fenced code 代码块写法也可以:</s>

github在5月1号后将不再支持[redcarpet]( https://github.com/vmg/redcarpet)解析器，建议使用[kramdown](http://kramdown.gettalong.org/quickref.html)，解决[办法](https://help.github.com/articles/updating-your-markdown-processor-to-kramdown)，按照教程在`_config.yml`文件中添加 `markdown: kramdown`即可（此主题的作者已修复这个问题）

所以使用 kramdown 代码块这样写就行

```
​```
CABasicAnimation *position = [CABasicAnimation animationWithKeyPath:@"position"];
​```
```



#### 本地预览

命令行进入博客目录，执行：

``` objectivec
jekyll server // 简写 jekyll s
```

在浏览器地址栏中输入：http://localhost:4000/ 就可以进行本地预览。新增、修改、删除文章都可以实时的看到，只需要刷新页面，可以试着修改默认那篇文章看看效果。

#### 发布文章

发布文章和提交git项目修改一样，基本流程大概是这样的，博客目录下执行

``` objectivec
git add .
git commit -m 'xxxxx'
git push origin master
```

完成之后应该就能看到新的文章已经在你的个人博客主页上了。



#### 替换谷歌字体库

<s>
网站打开慢？刚搭完博客我也在郁闷这个事，然后通过[google的网站速度分析](https://developers.google.com/speed/pagespeed/insights/)发现是JS加载google的字体库造成找到`/_layouts/default.html`，把下面代码中的Google免费字体库的域名`googleapis`替换成[360](http://libs.useso.com)提供的代理`useso`的
</s>

目前谷歌字体库已经可以正常使用，360貌似已经关闭了其字体库，所以不需要做任何的更改。

#### 配置博客的相关信息



在`_config.yml`文件中修改，比如我的配置如下

``` objectivec
name: Bawn
description: Blogging about stuffs
meta_description: "Bawn's Blog"

markdown: kramdown
redcarpet:
extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "tables", "with_toc_data"]

highlighter: pygments
logo: false
paginate: 20
baseurl: /
domain_name: 'http://bawn.github.io/'
google_analytics: 'UA-XXXXXXXX-X'

# Details for the RSS feed generator
url:            '/rss.xml'
author:         'Bawn'
```

#### 添加多说评论

将 `_layouts/post.html` 中的 `<footer class="post-footer">` 到 `</footer>`之间的内容替换为多说的评论代码，比如我替换后是这样的:

![image](/images/Jekyll/duoshuo.png)

不过多说代码中的`data-thread-key` 目前不知道怎么赋值。



#### 自定义代码高亮

如果不满意这个主题的代码高亮，也可以轻松定制，比如配置我现在用的高亮样式

1. 在[Bootstrap中文网](http://www.bootcss.com/) 提供的 [CDN](http://www.bootcdn.cn/highlight.js/) 链接找到你喜欢的样式链接，比如我用的是样式是`hopscotch`对应的是![image](/images/Jekyll/js.png)相应的样式效果可以到[highlightjs网站](https://highlightjs.org/static/demo/)查看，然后点击`复制<link>标签`
2. 找到`_layouts/default.html`并编辑，找到`<!-- Customisation  -->`代码，直接在下面粘贴刚才复制的代码，然后head里面看上去会是这样![image](/images/Jekyll/demo1.jpg)
3. 还是在`_layouts/default.html`，找到`<script type="text/javascript" src="/assets/js/index.js"></script>`在下方添加以下代码

``` objectivec
	<script type="text/javascript" src="http://cdn.bootcss.com/highlight.js/9.2.0/highlight.min.js"></script>
    <script type="text/javascript">hljs.initHighlightingOnLoad();</script>
    <style type='text/css'>
    .hljs {
      white-space: pre;
      word-wrap: normal;
      overflow-x: auto;
    }
    </style>
```



#### 最后

如果直接fork我的项目进行修改，那么自定的这些样式也会被保留，不用折腾替换谷歌字体库、自定义代码高亮之类的事情，但是必须注意以下几点

1. 在 `_layouts/default.html`底部有我的百度统计代码，请自行配置你自己的代码![image](/images/Jekyll/baidu.png)
2. 在 `_layouts/post.html`底部有我的disqus插件代码，请自行配置你自己的代码或者使用多说评论![image](/images/Jekyll/disqus.jpg)
3. 删除`_posts`文件下的所有文件
4. 修改`about.html`

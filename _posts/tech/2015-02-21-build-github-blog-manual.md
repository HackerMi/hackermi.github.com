---
layout: post
title: 使用Jekyll和GitHub Pages搭建博客原理、方法和资源
categories: tech
tags:  GitHub Git jekyll markdown blog 博客技术
---

## 一、缘起
&emsp;从大学到现在一直有记录自己生活的习惯，不过形式一直在变。大学时因为网易邮箱的缘故，顺便就用网易博客记录了。毕业后，工作一直比较忙，干脆用记事本偶尔写写。来北京后，尝试过新浪微博，新浪博客等。这些年折腾下来，基本算是业余博客写作爱好者。  
&emsp;早在13年就看到可以用github写博客，但因自己的懈怠，一直没有真正建起来。只是心血来潮的申请了一个好玩的域名，搞了一个测试页面，便荒废在那里。一转眼2年过去了，已是而立之年的我，貌似没有什么像样的东西。唯一的安慰就是自己还算努力，研究了不少东西，但疏于整理，一直荒废在电脑的某个文件夹里。新的一年打算给他们安个家，温故知新，输出一些有价值的东西。先从搭建GitHub博客开始，闲言少叙，开始正题。
## 二、基本概念
相较其他博客，GitHub上搭建博客有一定的技术门槛，需要自己动手摸索。对于不做前端开发的人来说，还是有些陌生。这里先说说自己对一些技术概念的理解。

<!--more-->

####[Jekyll](http://jekyllcn.com/)
官方的定义：将纯文本转化为静态博客网站。简单明了，第一眼感觉很是神奇。研究了一下他的工作原理，发现Jekyll其实有些像JAVA虚拟机。它有自己的编程语言Markdown 和 Liquid，用户利用这些语言为自己的blog“编程”，而后jekyll将他们翻译为HTML静态网页，在利用内置的HTML Server将静态网页运转起来，最终构成大家熟悉的网站。

####[Markdown](http://www.appinn.com/markdown/)
Markdown 有些像HTML语言，有自己的标记和符号。根据自己的使用情况来看，Markdown更多的是针对写作————简化排版操作、易用。它没有HTML语言那么多繁杂的标签，标签也更形象和人性化，用起来蛮顺手的。技术上，jekyll最终将Markdown翻译为对应的HTML语言，从这角度上说Markdown语言更像是在HTML语言之上做了进一步简化和包装，从而达到简单易用的目的。

####[Liquid](https://docs.shopify.com/themes/liquid-documentation/basics)
Liquid 是基于Ruby的模板语言，主要用来加载网站的动态内容。它有些类似javascript，主要帮助jekyll生成网站页面。相较Markdown方便用户写作的目的，Liquid则是方便用户控制自己的网站，包括布局，交互等。

####[GitHub & GitHub Page](https://github.com/)
GitHub 程序员圈大名鼎鼎的社交化代码管理网站，换句话就是Git的云版实现。有了它，程序员在任何有网络的地方随时提交代码，十分方便。GitHub 为每个用户提供了一种展示自己管理项目的机制 ———— GitHub Page，用户可以利用它编写自己的静态页面，说明自己的项目。同时静态页面也使用类似其他Git项目的代码管理。这不仅为用户提供了控制展示页面的权利，同时也免去了管理网站的麻烦。  
## 三、搭建思路
大家可能已经厌倦了各种技术术语的轰炸，这里换个角度介绍搭建思路。  
#### Blog 1.0 版本
我们从GitHub Page作为切入点。GitHub Page既然可以自己管理展示的页面，那我们就在本地构建一个静态网页构成的网站（即由静态网页和超链接组织起来的网站，本质上讲就是一些静态网页根据内容需要的重新组合），而后我们将它上传至GitHub，并将其首页作为GitHub Page的展示页面。理论上可行，但是这样做无疑增加了很多网站维护的工作，有办法简化吗？当然，来看2.0版本。
#### Blog 2.0 版本
对就是Jekyll，他可以将一般的文本转化为静态网站。我们用Jekyll编写自己的静态网站，来解决1.0中的网站维护问题。
Jekyll原生使用的是Liquid语言，用这个语言编写页面时还是需要处理一些页面的东西，写作的注意力右被分散了，能不能再简单点？当然，进一步3.0版本。
#### Blog 3.0 版本
这回Markdown登场了，简单看过Markdown的语法就会发现，这种语言就是专为写作设计的，完全不像HTML那么繁杂而且反人类。
Jekyll果断支持了这种语言，这样我们就可以把与写作无关的事减到最少。  

    总结一下整体思路：
    1. 使用Markdown撰写Blog内容
    2. 使用Jekyll生成静态网站页面
    3. 将内容提交GitHub，利用GitHub Page展示自己的Blog

## 四、搭建方法
搭建基于Github 的Blog主要包括两部分内容，本机环境以及网站配置 

###本机环境搭建  
本机环境主要用于写作、验证、排版内容，主要工作包括：  

    1. Git 本地环境安装  
    2. Jekyll 基础环境搭建
    3. Jekyll 安装
    
####Git本地环境安装
命令行版本请参照[官方文档](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)  
当然你如果不喜欢命令行，GitHub开发了图形界面的版本，请参考[官网](https://desktop.github.com/)
####Jekyll基础环境安装
Jekyll官方文档推荐通过Ruby Gem安装，所以需要安装Ruby开发环境，不同平台请参考：  
[Window平台安装Ruby](http://hustlei.github.io/2014/08/jekyll-install.html)，[Mac & Linux(Ubuntu)平台安装Ruby](https://ruby-china.org/wiki/install_ruby_guide)

    注意：
    1. Ruby 需要安装 v1.9.3 以上版本，旧版本的Ruby无法安装jekyll 2或3
    2. 有老版本的Ruby需要先卸载，再安装

有了Ruby开发环境后，接下来安装Ruby Gems —— Ruby的软件包安装工具，类似Ubuntu中的apt-get，安装过程请参考[官方教程](https://rubygems.org/pages/download)。 


最后是三个可选工具的安装，一个是NodeJS，安装请参考[官网](https://nodejs.org/en/)，它其实是JavaScript引擎，目前大部分系统都已经内置，一般没有特殊需要可以不安装；其二是Python，个人推荐还是安装，主要是为了代码高亮工具Pygments（安装和使用请参考[这里](http://havee.me/internet/2013-08/support-pygments-in-jekyll.html)）；最后是markdown渲染引擎，首选C语言编写RDiscount，内置的Maruku效率比较低，不推荐，安装方法参考[这里](http://havee.me/internet/2013-07/jekyll-install.html)

####Jekyll安装
准备好基础环境后，Jekyll的安装非常简单，只需一条命令：

    $ gem install jekyll

>Tips: 对于Window平台，整个安装也可以参考这篇[文章](http://cn.yizeng.me/2013/05/10/setup-jekyll-on-windows/)

###网站配置
网站配置主要用来发布内容，主要工作包括：

    1. GitHub账户注册
    2. Jekyll Pages页面的配置
    3. 独立域名的申请和挂接GitHub

####GitHub 账户注册
注册过程和一般网站没有区别，直接登录[官网](https://github.com/)注册即可

####Jekyll Pages页面配置
整个配置过程非常简单，主要步骤如下：

    1. 创建公开的代码仓库，使用username.github.io方式命名，其中username是你的github账户名 
    2. 同步代码库到本地
    3. 创建主页面（index.html）并提交到代码库

详细请参考[官网教程](https://pages.github.com/)

####独立域名的申请和挂接GitHub
说到域名申请首推Godaddy，国内的域名提供商不太靠谱。现在Godaddy支持支付宝了，购买域名也十分方便，这里找到一篇[百度经验](http://jingyan.baidu.com/article/a3aad71a80b3c4b1fa009649.html)不错，大家照做即可。  
    
    注意：网上有很多Godaddy优惠码，个人建议不要用，因为Godaddy后续会要求你上传身份证护照等信息所谓代价。

买完域名后，一般会找一个DNS服务商为你提供域名解析相关的服务，比如域名解析次数统计，域名安全等。当然这些不是必须的，不过国内有免费的域名服务商而且做的不错，而且帮你解决了电信，网通，教育网互通的问题，何乐而不为。这里推荐[DNSPod](https://www.dnspod.cn/), 配置很简单，只需添加一条UNAME记录即可：

 主机记录 | 记录类型 | 线路类型 | 记录值   |权重 | MX优先级 |	TTL
:--------:|:--------:|:--------:|:--------:|:---:|:--------:|:--------:
   www    |  CNAME   |  默认    |GitHub网址| -   |  -       | 600


详细教程参见[这里](http://jingyan.baidu.com/article/6181c3e04b5f2e152ef15321.html)。

最后，需要让GitHub知道你的域名，只需在项目的根目录下创建CNAME文件，并且填写域名即可，详细操作请参考[这里](http://jingyan.baidu.com/article/36d6ed1f5356f31bcf488314.html)

## 五、博客建设
这里主要整理一些有用的博客资源，以及日常使用技巧。

####1. Jekyll & MarkDown使用技巧

####1.1 常用的全局变量
<br/>

变量|描述
:--------|:--------
layout|如果设置了，则指定了所使用的布局文件，使用不带后缀名的布局文件名作为值。布局文件必须放在_layouts 目录中。
permalink|如果你希望当前文章的永久链接不同于默认的 /year/month/day/title.html 形式，则可设置该变量，它将作为最后生成文章的 URL。
published|如果你不想让某篇文章在生成的站点中出现，可将此变量设置为 false。
category/categories|可以指定一个或多个该文章所属的类别，可以以 YAML 列表的形式指定。
tags|与文章类别类似，也可以为文章指定一个或多个标签，同样适用 YAML 列表或空格分隔的字符串。

####1.2 发布草稿常用方法
* 在_post目录下的文章的YAML列表中增加 published 并设置为 false。
* 在根目录下创建_drafts目录，讲草稿文件放置在这个目录下使用jekyll s --drafts 讲_drafts目录下的文件加入网站编译，更多参考[这里](http://jekyllcn.com/docs/drafts/)

####1.3 [实现页内跳转的方法](http://www.cnblogs.com/JohnTsai/p/4027229.html#1.2)
  
####1.4 MarkDown的换行需要在行尾追加两个空格
  
####1.5 [MarkDown首行缩进的方法](http://metman.info/blog/2013/02/27/markdownru-men/)

####2. Blog 模板
  
* [Jekyll主题网站](http://jekyllthemes.org/)  
* [简洁明快风格](http://www.zhihu.com/question/20223939)  
* [优雅风格](http://www.zhanxin.info/themes.html)   

#### Markdown 免费编辑器
个人比较推荐在线编辑器Markable.in，所有平台都通用，云存储。

Windows 平台

* [MarkdownPad](http://markdownpad.com/)
* [MarkPad](http://code52.org/DownmarkerWPF/)

Linux 平台

* [ReText](http://sourceforge.net/p/retext/home/ReText/)

Mac 平台

* [Mou](http://mouapp.com/)

在线编辑器

* [Markable.in](http://markable.in/)
* [Dillinger.io](http://dillinger.io/)

浏览器插件

* [MaDe](https://chrome.google.com/webstore/detail/oknndfeeopgpibecfjljjfanledpbkog) (Chrome)

高级应用

* [Sublime Text 2](http://www.sublimetext.com/2) + [MarkdownEditing](http://ttscoff.github.com/MarkdownEditing/) / [教程](http://lucifr.com/2012/07/12/markdownediting-for-sublime-text-2/)

#### 博客评论工具
这里推荐Disqus，简单易用，详细教程参见[这里](http://blog.ihurray.com/blog/Disqus-learning.php)

#### Font Awesome 网站图标集与使用
[图标资源库](http://www.bootcss.com/p/font-awesome/)  
[图标使用教程](http://fontawesome.dashgame.com/#basic)

#### 网站CSS样式推荐使用Bootstrap，具体参见[官网](http://www.bootcss.com/)
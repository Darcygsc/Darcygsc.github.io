---
layout:     post
title:      "如何搭建自己的Blog "
subtitle:   "合格程序猿的基本条件之一"
date:       2015-08-12
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
---

**一个有追求的程序猿，一般都会有一个自己的个人博客。博客带来的好处有很多，比如可以记录自己成长的点点滴滴，可以培养自己的写作能力，可以将知识分享给需要的人。在帮助别人的同时，自己也会很有成就感--送人玫瑰手有余香。国内一些比较有名的iOS开发者都有维护自己的博客，像猿题库的[唐巧](http://blog.devtang.com/blog/archives/)，LINE的[喵神](http://onevcat.com/?from=inf&wvr=5&loc=infblog)，百度的[阳神](http://blog.sunnyxx.com/)等等。**

## 玩博客的话一般分三种情况
1.在一些大的博客网站注册，写自己的博客。如新浪博客、CSDN等。

2.自己写静态页面，选择一个免费空间来写。如github等。

3.发现免费空间限制太多，就自己购买域名和空间，搭建独立博客。如上述开发者。

这里介绍下第二种方式，选择用github搭建自己的个人博客。github简直就是程序猿的圣地。在github上搭建博客，使我们既拥有绝对管理权，又享受github带来的便利----不管何时何地，只要向主机提交commit，就能发布新文章。更妙的是，这一切还是免费的，github提供无限流量，世界各地都有理想的访问速度。

## 如何在github上搭建Blog

我们需要了解github的Pages功能，以及Jekyll软件的基本用法。

#### 一、github Pages 是什么？

github简单来说，它是一个具有版本管理功能的代码仓库，每个项目都有一个主页，列出项目的源文件。

![pages](http://7xl1kp.com1.z0.glb.clouddn.com/F414425E-E2FE-4748-8039-A2DB18A9F0E7.png)

但是对于一个新手来说，看到一大堆源码，只会让人头晕脑涨，不知何处入手。他希望看到的是，一个简明易懂的网页，说明每一步应该怎么做。因此，github就设计了[Pages](https://pages.github.com/)功能，允许用户自定义项目首页，用来替代默认的源码列表。所以，github Pages可以被认为是用户编写的、托管在github上的静态网页。

github提供模板，允许站内生成网页，但也允许用户自己编写网页，然后上传。有意思的是，这种上传并不是单纯的上传，而是会经过Jekyll程序的再处理。

#### 二、Jekyll是什么？

jekyll是一个静态站点生成器，它会根据网页源码生成静态文件。它提供了模板、变量、插件等功能，所以实际上可以用来编写整个网站。

#### 三、搭建环境
安装必备
1. Ruby：jekyll本身基于Ruby开发，因此，想要在本地构建一个测试环境需要具有Ruby的开发和运行环境。在windows下，可以使用[Rubyinstaller](http://rubyinstaller.org/downloads/)安装。(Mac OS X 10.5以上都自带)
2. RubyGems：目前新版的Ruby自带gem了，所以gem安装可以跳过。(Mac OS X 10.5以上都自带)
5. git：通过http://sourceforge.net/projects/git-osx-installer/ 下载安装即可(Mac OS X 10.5以上都自带)

然后使用命令行安装Jekyll

```
sudo gem install jekyll
```
如果希望加快应用程序的下载速度，特别绕过“天朝”的网络管理制度，可以选择国内的仓库镜像，taobao有一个：http://ruby.taobao.org/。

```
// 移除官方镜像源
gem sources --remove https://rubygems.org/
// 添加淘宝镜像源
gem sources -a http://ruby.taobao.org/
```

验证替换是否成功，在命令行输入 **gem sources -l**
出现

```
*** CURRENT SOURCES ***

http://ruby.taobao.org/
```

说明已经替换成功

#### 四、安装主题
1. Fork我使用的这款主题[Clean](https://github.com/IronSummitMedia/startbootstrap-clean-blog-jekyll)，也可以使用[这些模板](http://jekyllthemes.org/)，当然如果有能力也可以自己写主题。
2. 修改这个项目的名称为：xxxxxx.github.io。xxxx为你的github用户名，比如我的用户名是loveace，那么就需要修改成loveace.github.io，这个也正是你博客的地址。
![rename](http://7xl1kp.com1.z0.glb.clouddn.com/E7E6843C-33C5-451B-B6B0-42E19DE667E4.png)

完成这两步之后，在你的浏览器上输入xxxxxx.github.io或者xxxxxx.github.com，就会出现你个人博客的页面(可能需要等待几分钟)，这时候你的博客上 应该有一篇主题作者的默认文章叫做Welcome to Jekyll!

#### 五、发表文章
首先我们需要将github上刚才fork的资源下载到本地，在命令行输入

```
// 进入主题需要放置的目录
cd Documents
// 克隆刚才fork的主题
git clone https://github.com/xxxx/xxxxx.github.io
```

完成之后你的Documents目录下应该有个xxxxx.github.io的文件夹（称之为博客目录），打开文件夹可以看到。

![file](http://7xl1kp.com1.z0.glb.clouddn.com/2DAB18E2-21B8-4C4F-A5AF-1AA0458D518F.png)

_posts文件夹中的markdown文件就是所发表的文章的源文件。其它文件的意义可以自行研究。
新建一篇文章的命名规则是xxxx-xxx-xx-xxxxxxx.md，比如我这篇文章的命名是2015-8-12-createBlog.md，为了能够呈现出想要的样式，需要在文件头部加入以下代码:

```
---
layout:     post
title:      "如何搭建自己的Blog "
subtitle:   "合格程序猿的基本条件之一"
date:       2015-08-12
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
---
```

具体编写文章，这里使用的是markdown工具，可以查看下markdown的使用[语法](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown)，在编辑图片的时候，一般采取使用外链的形式。可以事先将文章存到入[七牛云](https://portal.qiniu.com/)中，然后将外链地址复制到markdown中即可。

Jekyll提供了本地预览功能，命令行进入博客目录，执行：

```
jekyll server
```

在浏览器地址栏中输入：http://localhost:4000/ 就可以进行本地预览。新增、修改、删除文章都可以实时的看到，只需要刷新页面,可以试着修改默认那篇文章看看效果。

在写好文章之后，发表文章可以进入博客目录下，在命令行输入:

```
git add .
git commit -m 'xxxxx'
git push
```

命令输入完成，界面如果显示
![success](http://7xl1kp.com1.z0.glb.clouddn.com/114B0286-C809-4F60-B3B5-BE7B5C233598.png)
表示提交成功。

成功提交之后，刷新网页，就可以看到你最新发表或者修改的文章了。
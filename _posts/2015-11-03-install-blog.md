---
layout: post
title: 如何搭建自己的github博客
category: personal
modified: 2015-11-03
tags: [personal,blog]
comments: true
pinned: true
excerpt: "准备工作(windows环境)1. Git for Windows2. MarkdownPad 2 3. 一下午或者几个小时时间 技能需求1. git 基本命令2. 基本的HTML等前端知识3. 有一种维修工的嗅觉..."
---

#准备工作(windows环境)
1. Git for Windows
2. MarkdownPad 2
3. 一下午或者几个小时时间


#技能需求
1. git 基本命令
2. 基本的HTML等前端知识
3. 有一种维修工的嗅觉


#开工
　　首先安装好git 和makdownpad，百度一下就知道怎么安装了，很简单就不给出地址了，因为很多人使用git的工具不同，看自己的喜好了。  
　　我使用的是jekyll搭建的，不需要安装任何额外的程序或者文件，你要是想炫酷一点当然是自己制作、自己修改的比较好，但是如果你跟我一样是个不喜欢折腾前端界面的CODER，那么恭喜你，你肯定很喜欢直接修改字段和字符串就好了。  
1. [jekyll基本介绍](http://www.chinaz.com/web/2014/0616/355745.shtml)
2. [jekyll进阶](http://higrid.net/c-art-blog_jekyll.htm)
3. [一些模板](http://www.zhihu.com/question/20223939)   [再来一些](http://jekyllthemes.org/)
4. [最好还是看一些jekyll的官方介绍](https://github.com/jekyll/jekyll)  
好了差不多你也了解很多的jekyll和github的知识了。下面你需要找到你看中的github上的blog，[当然你需要fork我的blog也可以在这里，请点一个星，谢谢，在右上角Star](https://github.com/ch710798472/just_fork_blog)找到一个然后fork到自己的github（右上角的fork按钮），然后改工程名为blog（改工程名是要进工程，然后在左下角有个setting，点进去，最上面有个RENAME，对就是它了），接下来需要在本地的电脑上操作了。
```
mkdir blog
cd blog
git init
git checkout --orphan gh-pages
git pull git@github.com:YourGithubName/blog.git
在这一步尽情的修改你的文件
git remote add origin git@github.com:YourGithubName/blog.git
git add .
git commit -m "update"
git push origin gh-pages
```  
首先你需要修改的是_config.yml,基本上不用动，只需要该url和你自己的名字和宣言啥的，别乱改，不知道咋改的就不要改，先改`title:  email:  url:   name:  avatar:  disqus-shortname:  github:  weibo:`这几个字段，改成你自己的东西。  
##disqus
什么你要问disqus-shortname是啥，自行百度下，使用还是简单的很，首先得注册一个disqus账号，并在disqus里面将你的域名添加进去，在添加成功的时候会提示你有一个shortname，写进这里就行了。
![](http://i.imgur.com/C6CNAz2.png)
![](http://i.imgur.com/HlTTUuW.png)
![](http://i.imgur.com/imwrdVg.png)
![](http://i.imgur.com/7O4Nxzl.png)

#结果  
　　好了，到这里你的github博客应该可以访问了，YourGitName.github.io/blog就能访问了,如果不能访问，请在新建一个YourGitName.github.io的项目，可能是没有这个项目导致的，你说为什么不用YourGitName.github.io作为域名，我这不是留给你自己写自己真正的博客用的嘛，我只是给你个参考而不能让你用不了本身的YourGitName.github.io。
![](http://i.imgur.com/bb7e5Qc.png)  
　　如果你觉得这篇博客还不错，请给个评论或者给我的github项目一个星[myblog](https://github.com/ch710798472/just_fork_blog)。谢谢，我会在空闲的时间更新个友情链接的模块，到时候大家可以把我得博客加入到友情链接里面。


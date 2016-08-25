---
layout: post
title: 让你的ubuntu美起来
category: ubuntu
modified: 2015-12-02
tags: [ubuntu,desktop]
comments: true
pinned: true
excerpt: "1#不用任何附加工具，安装ubuntu14.04双系统 2#谷歌浏览器 3#搜狗输入法 4#桌面图标和主题美化..."
---

#　　不用任何附加工具，安装ubuntu14.04双系统
　　百度ubuntu官网，下载你想要的ubuntu版本，以下优化只能是14.04以上的版本，所以不要下12.04了。下载完之后，在安装个能打开iso镜像文件的UItraISO软件，百度安装即可，不用注册直接试用，然后打开你的ubuntu ISO后缀文件，发现里面有个wubi.exe程序，直接拖出来就行，不能把整个ISO解压了，放在跟ISO文件同级的目录下，直接运行wubi.exe程序。你发现很简单，设置下安装盘，最好别放在C盘，用户名密码随便填最好设置比较简单，不然每次sudo的时候输入密码就很费劲，桌面选择GNOME，其他的都不要动了，下一步下一步，然后重启。选择ubuntu系统，确定，进入到自动拷贝文件
　　安装模式，在自动重启，但是这里有个BUG需要解决在你正式进入到ubuntu系统，不然你也是进不去的。
　　1. 第二次重启开机之后，进入选择 ubuntu 启动的画面（ubuntu and Advance）。按键盘E，进入编辑。找到“ro rootflags=sync”，改为“rw rootflags=sync”。再按F10启动。
　　2. 进入系统后，打开终端，执行
`$sudo gedit /etc/grub.d/10_lupin`
改动文件的第 150 行，把ro改成rw，保存。
```
linux ${rel_dirname}/${basename} root=${LINUX_HOST_DEVICE} loop=${loop_file_relative} ro ${args} //修改前
linux ${rel_dirname}/${basename} root=${LINUX_HOST_DEVICE} loop=${loop_file_relative} rw ${args} //修改后
```
然后更新启动器的配置文件
`$sudo update-grub`

###　　NOTE：首先别把更新源（软件和更新）给禁止了，最少要留下重要更新等，不然很多软件安装失败，弄完了在优化系统。(安装之后的优化)[http://tieba.baidu.com/p/3000822153]，一般不需要这个步骤，你可以跳过之后再看。

#　　谷歌浏览器
[http://www.linuxidc.com/Linux/2014-04/100645.htm](http://www.linuxidc.com/Linux/2014-04/100645.htm)

#　　搜狗输入法
[http://jingyan.baidu.com/article/ad310e80ae6d971849f49ed3.html](http://jingyan.baidu.com/article/ad310e80ae6d971849f49ed3.html)
装好之后别忘记右键点击右上角的搜狗图标，换个皮肤。


#　　桌面图标和主题美化
[http://www.cnblogs.com/youxia/p/linux012.html](http://www.cnblogs.com/youxia/p/linux012.html)
我用的是最好看的那个，你问我哪个最好看，这需要质疑吗？当然还需要利用Unity Tweak Tool调节透明度和任务栏的颜色等，自己琢磨琢磨吧，不想琢磨可以跳过。

#　　标题栏实时显示网速CPU等
[http://tieba.baidu.com/p/3005287033?see_lz=1](http://tieba.baidu.com/p/3005287033?see_lz=1)

#　　conky manage
如果你不想折腾，安装这个吧，比较简单，修改的话参照下面的配置信息解释，自己改一改
[https://linux.cn/article-3248-1.html](conky manage)

#　　conky
喜欢自己折腾就用这个，当然了，想要好看的去ubuntu社区看看发布的.conkyrc文件吧
[http://www.linuxidc.com/Linux/2012-09/71478.htm](conky)

#　　conky天气解决方案：
[http://blog.163.com/yogi_nore/blog/static/22017305201002633734971/](http://blog.163.com/yogi_nore/blog/static/22017305201002633734971/)
[http://tieba.baidu.com/p/577532690?see_lz=1](http://tieba.baidu.com/p/577532690?see_lz=1)

#　　conky具体配置信息解释：
[http://tieba.baidu.com/p/1996230250](http://tieba.baidu.com/p/1996230250)
[http://www.ourys.com/post/60.html](http://www.ourys.com/post/60.html)

#最终大杀器－－据说比苹果系统还好看的界面cairo-dock
　　[http://jingyan.baidu.com/article/4853e1e519e70c1908f7266e.html](http://jingyan.baidu.com/article/4853e1e519e70c1908f7266e.html)
   我们安装成功之后可以在最下面的工具栏单击右键，选择自己喜欢的插件，其中天气是最实用的，不过可能地址不对，你需要找到插件设置里面的天气官网，是国外的网站，进去之后搜索自己的地区，把代号填到插件设置里面去，就可以了。
![](http://i.imgur.com/ifJc4tj.jpg)
![](http://i.imgur.com/ORY8654.jpg)
![](http://i.imgur.com/hn6BVqg.png)
![](http://i.imgur.com/PFbuqng.png)
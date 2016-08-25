---
layout: post
title: 安卓安全（逆向apk）
category: security
modified: 2016-01-08
tags: [security,apk,smali]
comments: true
pinned: true
excerpt: "1. 逆向给定的两个apk，SimpleCrackme，Crackme01）2.找到正确的key（正确的key输入后点击check按钮会出现“黄色背景的提示文字”，错误的输入是“灰色背景的提示文字”）..."
---

#Lab4 OverView
　　利用工具破解需要输入特定字符串的apk，要求
1. 逆向给定的两个apk 
    Tips：难以程度[由易到难]：SimpleCrackme，Crackme01）
2. 找到正确的key（正确的key输入后点击check按钮会出现“黄色背景的提示文字”，错误的输入是“灰色背景的提示文字”）
3．独立完成，方法不限
4. 实验环境
- windows7
5. 实验工具
- baksmali.jar
- smali.jar
- signapk
[实验所需全部工具源码](http://pan.baidu.com/s/1jHw2k3G) 密码：b4tg

#开始解题
#### 1. 先来破解SimpleCrackme.apk，解压apk文件得到classes.dex文件，利用baksmali.jar得到smali代码，如下图所示（out文件夹内）：
![](http://i.imgur.com/fNOdU0p.png)

####然后阅读smali代码可知，在a.smali文件中有处理字符串检查的关键的方法，所以仔细阅读了a.smali代码，找到了用于比较的字符串，然后输入就能破解了。很简单。
![](http://i.imgur.com/aTNwbab.png)
####当然也可以修改smali代码使得检查字符串的方法返回值永远是1，可以这么修改：
![](http://i.imgur.com/9Z7eEoc.png)
####然后用smali.jar打包成.dex文件，复制到原来的解压文件下，然后在压缩文件，改后缀为apk，在用signapk重新签名，安装运行。
![](http://i.imgur.com/xG0I6uC.png)
-----------------------------------------------------
![](http://i.imgur.com/q49hpG0.png)
-----------------------------------------------------
![](http://i.imgur.com/AAo6Lay.png)
####结果如下（一个是破解字符串，一个是修改了源码的）：
![](http://i.imgur.com/CQMGoLC.png)
-----------------------------------------------------
![](http://i.imgur.com/asWwuiq.png)
-----------------------------------------------------
####2. 破解Crackme01.apk，首先同样的利用baksmail.jar反编译classes.dex得到我们能看得懂的smali代码，然后阅读代码，发现a.smali中用于检查字符串的代码用的是invoke-virtual {v2, v3}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z，而且返回值是布尔类型，而且文件里也没有声明字符串，那么只需要把v2打印出来看看就知道是什么了（不打印v3的原因是从它是MainActivity传入的参数，iget-object v3, p0, Lcom/exiahan/crackme01/a;->c:Ljava/lang/String;），在字符串比较之前加入一段代码用于在控制台输出字符串：
![](http://i.imgur.com/urNP53S.png)
####当然了，仅仅加这个代码会因为签名的改变导致程序不能正确运行，那么在MainActivity.smali找到用于程序中签名的函数，改变其返回值即可。
![](http://i.imgur.com/Q7DD6k5.png)
####在打包成dex文件然后在重新签名，安装程序：
![](http://i.imgur.com/inbEDsv.png)
####我们打开程序，随意输入然后点击按钮，会发现控制台输出了key，接着把key输入输入框，即可破解了。
![](http://i.imgur.com/0aIWH1t.png)
![](http://i.imgur.com/oEbl6Sd.png)
####而且跳过输入字符串也比较简单，只需要修改a中相应方法的返回值即可，也可破解.
![](http://i.imgur.com/40Xo2MJ.png)
![](http://i.imgur.com/Yedj3kq.png)

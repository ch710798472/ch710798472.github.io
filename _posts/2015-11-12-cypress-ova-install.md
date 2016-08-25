---
layout: post
title: 基于edustack.org上的cypress ova版本的edX安装
category: edx
excerpt: "首先我们在mirrors.edxstack.org下载所需要的ova后缀结尾的文件..."
modified: 2015-11-12
tags: [edx,cypress ova]
comments: true
pinned: true
---
#准备工作

宿主机环境
1. win7
2. virtual box 4.3.24版本

#安装
　　用vbox打开你下载的[.ova](http://mirrors.edustack.org/)文件，然后成功的导入之后（怎么导入ova请自行百度），启动虚拟机，到目前为止不需要修改任何配置。如果不出意外的，你会看到黑屏幕上有个openedx的图案，这是tty界面，这时候需要输入用户名密码，用户名：edustack 密码：edustack.org
![](http://i.imgur.com/kf6DFST.png)
###　　一、修改网络配置
　　在virtualbox上找到设备->网络->更改网络配置，进入之后修改为桥接模式，界面名称这个你可以试试，总共没多少个，我是选的Network Adapter结尾的一个，点击确定。你的openedx会出现一行命令，不用管，确定即可。下面修改配置文件
![](http://i.imgur.com/IKctvXc.png)

```
sudo vi /etc/network/interfaces

...
address 192.168.2.199
netmask 255.255.254.0
network 192.168.2.0
broadcast 192.168.2.255
gateway 192.168.2.1
dns-nameservers 114.114.114.114
```
　　说明一下，之所以选择桥接模式，是因为可以用你的电脑直接在浏览器里面输入你的虚拟机地址加上端口号，就可以访问edx了，当然这是对于开发者来说的，如果你要布置生产环境，还需要配置网址等等。address 就写你可以获得可用的IP地址，比如你的IP地址是192.168.1.100，那么如果你的路由器是动态配置的话你就可以写192.168.1.150等等，broadcast就跟你的IP保持一致，最后一位一定是255。子网掩码和网关你得查看你的电脑网络配置。运行cmd，输入ipconfig，得到以上信息写入，DNS就写4个114就可以了。重启虚拟机网络：

```
sudo /etc/init.d/networking restart
在看看网络设置成功没
ping www.baidu.com
```
　　没成功，你需要换个界面名称，然后在重启一下网络。因为我不太懂界面名称的不同对于网络的影响，所以你得靠自己了。多尝试，肯定可以的。
　　然后你可以打开浏览器输入网址打开lms->虚拟机ip:80,cms->虚拟机ip:18010
![](http://i.imgur.com/S9bUDgG.png)  

　　这是一些测试账号，格式是账号功能 账户名:密码

```
Staff staff@example.com:edx
Honor honor@example.com:edx
verified verified@example.com:edx
audit audit@example.com:edx  
```

　　如果你的要求不只是开发调试，请自己参照下面配置。[edustack.org网站](http://edustack.org/manual/edx/ova%E9%85%8D%E7%BD%AE%E6%8C%87%E5%8D%97/)

###　　其他配置
　　其他相关的配置，请参考[edustack.org网站](http://edustack.org/manual/edx/ova%E9%85%8D%E7%BD%AE%E6%8C%87%E5%8D%97/)  
　　不过可能，某些命令不太适用了，比如汉化就可以不用了，自带汉化了，个别地方没有完成的汉化可以自己改一下。创建超级用户命令不适用了，具体可以参考如下：

```
sudo -u www-data /edx/bin/python.edxapp ./manage.py lms --settings aws create_user -s -p edx -e user@example.com

sudo -u www-data /edx/bin/python.edxapp ./manage.py lms --settings aws changepassword user

sudo -u www-data /edx/bin/python.edxapp ./manage.py lms --settings aws shell

from django.contrib.auth.models import User
me = User.objects.get(username="user")
me.is_superuser = True
me.is_staff = True
me.save()

解释：./manage.py 是edx-platform下的
```

　　其他的管理命令，请照例修改。另外在输入命令过程如果碰到密码错误或者不知道的密码，证明你用户账号弄错了，只有在edustack下使用密码edustack.org才是正确的，你需要退出当前账号，在输入命令。
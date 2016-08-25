---
layout: post
title: 基于docker的open edX安装
category: edx
excerpt: "1. 安装使用docker2. 拉取docker镜像文件3. 运行edx..."
modified: 2015-11-26
tags: [edx,docker]
comments: true
pinned: true
---


#安装环境 ubuntu14.04
###1. 安装使用docker

```
curl -sSL https://get.daocloud.io/docker | sh
```

###2. 拉取docker镜像文件,我们使用的是种瓜大神的版本[https://hub.docker.com/r/wwj718/edx_cypress_docker/](https://hub.docker.com/r/wwj718/edx_cypress_docker/)，最后是版本号，如有更新请改成你想要的版本,大概下载会有2G多一点，但是解压之后有5G多一点，但是由于种瓜大神当时命名错误，所以会拉取两个版本，所以剩余空间要10以上才能完成安装，可能之后会修复的，那就不用安装完成之后在删除了其中一个了，最后会告诉你怎么删除。

```
sudo docker pull wwj718/edx_cypress_docker:1.21
```

###3. 运行edx，其中的端口映射2022是ssh端口，80和18010分别是lms和cms的页面（完全开启大概需要十几秒，端口冲突请随机分配端口sudo docker run -itd -P wwj718/edx_cypress_docker:1.21）。
```
sudo docker run -itd -p 80:80 -p 2022:22 -p 18010:18010 wwj718/edx_cypress_docker:1.21
```
成功之后会输出一段数字字母字符串

###4. SSH登陆到EDX里面去，用户名root密码edx

```
ssh root@localhost -p 2022
```
![](http://i.imgur.com/LSjenoJ.jpg)

###5. 使用Pycharm开发的话，只需要把remote Python interpreters指向docker中的/edx/app/edxapp/venvs/edxapp/bin/python即可

###6. git clone平台代码，比如用种瓜大神的`git clone -b edx_cn/cypress_cn https://github.com/easy-edx/edx-platform.git ~/edx-platform`

###7. 如果是开发的时候不需要运行第3步，直接运行开发环境即可，-v 后面是你想要挂在的目录以及映射到docker里面的目录，值得说明的是，edx的所有代码在根目录下，不要ssh进去之后用命令ls查看，没有代码就认为安装失败了，请直接进入/edx（根目录下）目录查看即可。

```
sudo docker run -itd -p 5000:5000 -p 5010:5010 -p 2022:22 -v ~/edx-platform:/edx/app/edxapp/edx-platform-dev wwj718/edx_cypress_docker:1.21
```
###结束之后，记得保持良好的习惯关闭容器，避免导致不必要的错误（sudo docker stop id）。

###8. 编写完代码启动cms和lms是在/edx_dev目录下有start_lms和start_cms直接运行即可，打开5000,5010端口即可看到lms和cms。代码的保存是在你的本机上而不是虚拟机里面，所以请自行的做好备份（用github）。

###9. 每次不必重新启动一个容器，查看所有容器`sudo docker ps -a`然后重启你需要的容器`sudo docker start id`(id就是之前查看的容器的ID一串数字)，这样做的好处是你想要删除某些镜像的时候不必清除冗余的一些依赖。

###10. docker的一些有用操作
```
查看容器运行的程序
sudo docker top id

查看容器运行的程序

sudo docker top id

查看历史记录命令
sudo docker history id

终止运行的容器
sudo docker kill/stop id

删除镜像或者容器
sudo docker rm/rmi id
```

###11. 登陆到edxapp并且加载环境（这些都不太需要，只是演示如何登陆其他用户）
```
sudo -u edxapp /bin/bash
source /edx/app/edxapp/edxapp_env
paver devstack [--fast] lms/studio
```

###12. 如果启动容器的时候碰到错误，提示无法连接容器，请重启docker服务或者重启机器。

##13. 最好看一看熟悉一下流程。分工协作，或者开发一个XBLOCK请参考[种瓜大神的博客](http://wwj718.github.io/install-youkuXblock-into-edx.html)

##14. 如果是windows环境，请找到你的docker的虚拟机的IP替换上面的localhost


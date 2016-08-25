---
layout: post
title: 信息安全（return-to-libc解答）
category: security
modified: 2015-11-07
tags: [security,return-to-libc,ustc]
comments: true
pinned: true
excerpt: "Exercise 1：把第一次试验中的攻击代码写到stack2.c文件中，编译运行，发现不能成功的攻击，会报出段错误的提示，如图下图所示。..."
---
#Exercise 1：
####　　把第一次试验中的攻击代码写到stack2.c文件中，编译运行，发现不能成功的攻击，会报出段错误的提示，如图下图所示。 
![](http://i.imgur.com/QbQocHm.png)

---
#Exercise 2:
  首先照着实验指导一步步的先自己gdb调试，具体的调试如下图所示：
![](http://i.imgur.com/sEvuCvq.jpg)
####　这么做之所以会收到一个段错误，是因为没有在里面放入exit函数的地址。应该放到ebp+8的位置，紧跟在system函数地址的后面，当作system函数的返回地址。正确的做法如下图所示：
![](http://i.imgur.com/5tC5DaL.png)

---
#Exercise 3:
####　　要想攻击服务器必须的找到想要攻击的s字符串数组的首地址，以及fd的地址，因为&fd-s的值就是s数组距离当前堆栈的函数的第一个参数的距离，这个值能很好的帮我们判断到底在哪里写入攻击的代码。首先在parse.c文件中写入一行代码来输出s以及fd的地址printf(“s=%p,fd=%p\n”, s, &fd)，如下图所示：
![](http://i.imgur.com/lEjA97Q.jpg)
####　　我们计算出&fd-s的差值是1064，也就是说&fd-4就可以存放攻击代码了，我们可以构造一个system+exit+”/bin/sh”的攻击代码。也就是说在browser.c中我们传入服务器的数据的数组req[1060]中存放system函数的地址，req[1064]中存放exit函数地址，req[1068]中存放想要实现的功能的字符串地址。
![](http://i.imgur.com/rTrFtXQ.jpg)
####　　但是实际的攻击并没有如我们预期那样出现结果，而是出现了一个莫名其妙的shell指令，
![](http://i.imgur.com/0HnzVu0.png)
####　　而且各个函数和字符串的地址都是正确的，经过gdb调试辨认可知莫名其妙的指令是在实际的SHELL=/bin/bash之前的几个字节，下图为调试内容。
![](http://i.imgur.com/kkOL3Jj.png)
####　　经过上次的实验可知，需要在服务器运行的时候进行调试，才能得到正确的地址，所以调试如下，最终找到正确的地址取得权限。
![](http://i.imgur.com/YUKjKDW.png)

![](http://i.imgur.com/zUtRTHJ.png)
####　　除了取得权限之外，我还尝试了让服务器打开一个xterm命令窗口，如下图所示，攻击代码有效了，在xterm窗口中可以看出是在服务器所在文件打开的xterm。
![](http://i.imgur.com/4rv0lQB.png)
#Challenge:
####　　通过export MYSHELL=/bin/sh，来写入环境变量，然后通过函数getenv()来取得地址(或者直接gdb调试查看，环境变量的值总是存在靠近0xbfffffff附近的位置)，新建一个getenvaddress.c文件，内容是：
```
void main()
{
 	　char* shell = getenv("MYSHELL");
  if(shell)
      printf("%x\n", (unsigned int)shell);
}
```
####　　取得了环境变量的地址如下，然后取得地址之后攻击跟上面的没什么区别，很好做了。
![](http://i.imgur.com/NEZJj7t.png)

---
#Exercise 4:
####　　启动Address Space Layout Randomization之后再进行攻击，结果如下图所示：
![](http://i.imgur.com/Iw1WqWn.png)
####　　攻击并不能够成功，而且地址每次运行都会变（每次重启之后），具体的值已经在程序里面输出了，不能被利用了。如下图所示：
![](http://i.imgur.com/lp1Qlt9.jpg)

---
#Exercise 5:
####　　由于我们只能猜测地址，因为程序执行实在用户态，所以地址范围只能是从0x00000000到0xbfffffff，但是由于堆栈是在内存顶部靠近3G的区域，所以基本上从0xbf000000开始比较好。我只需要修改browser.c文件中发送字符串的部分，并且加上一个循环，使得地址值从0xbf000000开始自增就可以了。

#实验总结：
####　　从上次的缓冲区溢出的实验，到这次的库函数攻击实验，都是要我们必须了解函数调用时候的堆栈的情况，对于内存的操作要准确，有一点点的偏差就会导致实验不成功，由于比上一次积累了更多的经验，所以这一次的实验比上一次的轻松很多，遇到问题的时候多请教同学和助教学长，多百度，总是会有解决掉方法的。




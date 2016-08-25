---
layout: post
title: 信息安全（缓冲区溢出攻击）
category: security
modified: 2015-11-03
tags: [security,buffer overflow]
comments: true
pinned: true
excerpt: "In this lab, you’ll understand the principal of buffer overflows and will understand how such attacks happen in real-world application (say, a web server)..."
---

#Lab1 OverView

##Important Note:   
This course’s labs, including this lab, ask you to design exploits and to perform attacks. These exploits and attacks are realistic enough that you might be able to use them to perform a real-world attack, but you should not do so. The only goal of the designing exploits is to teach you how to defend against them, not how to use them to attack others—-attacking computer systems is illegal and can get you into serious trouble. Don’t do it.

In this lab, you’ll understand the principal of buffer overflows and will understand how such attacks happen in real-world application (say, a web server);

##Lab Environment Setup

Download the [SEED-Ubuntu12.04](http://jupiter.syr.edu/seed/images/12_04_v2/SEEDUbuntu12.04.zip), open it with VirtualBox or VMWare. Don’t do the exercises on your bare machine, it may bite yourself.
Note: Accounts in the VM
ID: root Password: seedubuntu
ID: seed Password: dees
Download the [lab1-code](http://pan.baidu.com/s/1pJKPPjX)
##Lab Requirement

There are two kinds of exercises: normal exercises and challenge ones. Challenge exercises may not be that hard, but may involve substantial code hacking. You are required to do ALL normal exercises. All challenge exercises are optional (but you’re encouraged to try them).

##Hand-in Procedure

When you finished your lab, zip you solutions and submit to the school’s information system.
Note:
Your submit should include all the questions asked in each exercise.

#Part A: Buffer Overflow Principal

In this part of the lab, you will study the basic principal of buffer overflows, and then you will study how to use buffer overflows to attack a simple vulnerability program, some basic theorem will assistant you to realise the goal.

Now, unzip the lab1-code.zip, browse the source code we given and find out the file stack1.c. There is a simple C program in this file, which has buffer overflow vulnerability. You can compile this program using the following command:
```
$ make stack1
$ ./stack1
```
##Note:   
View the Makefile, you should know why the command make stack1 can compile the source file stack1.c, and be careful of the -g option , which will be useful when you debug the executable file with gdb.

###Stack layout and Buffers

In computer science, a call stack **is a stack-like data structure holding information to control function calls and returns. Stack layout is the convention on how the stack frame is used. As an example, read the simple C program given you (the C file stack1.c).
When the function main calls the function func, the stack layout looks like the following, pay special attention to the positions of the local variables and arguments.

                     +------------------+ high address
                     |       ...        |
                     |  stack frame of  |
                     |   main           |
                     |       ...        |
                     +------------------+
                     |  str(a pointer)  | (4 bytes)
                     |  return address  | (4 bytes)
           %ebp----> |    saved %ebp    | (4 bytes)
                     +------------------+
                     |       buf[11]    |
                     |       ...        | (12 bytes)
            buf----> |       buf[0]     |
                     |     variable_a   | (4 bytes)
                     |       ...        |
                     +------------------+ low address
##Exercise 1.

Now, you can write some code. Your job is to print the address of the variable buffer, in the C program stack1.c, and compile the C program as above.
Run it three times, observe and write down the output addresses in address.txt, are these 3 addresses the same or not?
Challenge!

Read the file /proc/pid/maps on your machine (pid is the process id), observe the value of [stack].
##Note:
You can read this article to learn how effective the ASLR on Linux systems is.

Now you can investigate the stack layout and C calling convention in detail, for this, you should use the debugger gdb. In this and future labs, you will use gdb heavily.

##Exercise 2.
Use gdb to debug the program, as the following. You may find the online gdb documentation useful.
```
$ gdb stack1
(gdb) b  func
Breakpoint 1 at 0x8048412: file stack.c, line 8.
(gdb) r
Starting program: /tmp/stack1
8      strcpy(buffer,str);
(gdb) info r
eax            0x80484e8    134513896
ecx            0xbffff504    -1073744636
edx            0xbffff494    -1073744748
ebx            0xb7fc8000    -1208188928
esp            0xbffff410    0xbffff410
ebp            0xbffff438    0xbffff438
esi            0x0    0
edi            0x0    0
eip            0x8048412    0x8048412
eflags         0x282    [ SF IF ]
cs             0x73    115
ss             0x7b    123
ds             0x7b    123
es             0x7b    123
fs             0x0    0
gs             0x33    51
(gdb) x/2s 0x80484e8
0x80484e8:     "I am greater than 12 bytes"
0x8048503:     ""
(gdb) p &buffer
$1 = (char (*)[12]) 0xbffff424
(gdb) x/4wx 0xbffff424
0xbffff424:    0x08048320    0x00000000    0x080482bd    0xb7fc83e4
(gdb) x/8wx $ebp
0xbffff438:    0xbffff468    0x08048443    0x080484e8    0xbffff504
0xbffff448:    0xbffff50c    0xb7e54225    0xb7fed280    0x00000000
(gdb) x/2i 0x08048443
0x8048443 :    leave
0x8048444 :    ret
(gdb) disass func
Dump of assembler code for function func:
0x0804840c :    push   %ebp
0x0804840d :    mov    %esp,%ebp
0x0804840f :    sub    $0x28,%esp
0x08048412 :    mov    0x8(%ebp),%eax
0x08048415 :    mov    %eax,0x4(%esp)
Address Space Layout Randomization
```

In order to protect against buffer overflows, most recent operating systems introduce many protection mechanisms, among which the most important one is address space layout randomization (ASLR). Basically, in a system with ASLR, the starting address of the heap and the stack, along with other segments, will be randomized, so it’s will be difficult for the attack to know or guess the specific address of any memory segments, say the stack. Here is a brief introduction, in lab 2, you will study ASLR in detail and learn how to defeat ASLR. For the purpose of this lab, you should simply turn off ASLR (in lab 2, you’ll perform attacks when ASLR is effective), which will make your attack easier to achieve. To turn off ASLR, you can run these commands:
```
 $ su root
 Password : (enter root password)
 # sysctl -w kernel.randomize_va_space=0
```
##Exercise 3.

Turn off the address space layout randomization, and then do exercise 1 again, write down the three addresses in args.txt, are those three addresses same or not?
Buffer Overflow and Shellcode

A buffer overflow occurs when data written to a buffer exceeds the length of the buffer, so that corrupting data values in memory addresses adjacent the end of the buffer. This often occurs when copying data into a buffer without sufficient bounds checking. You can refer to Aleph One’s famous article to figure out how buffer overflows work.

Now, you run the program stack1, just like below.
```
$ ./stack1 aaaaaaaaaa
Returned Properly
$ ./stack1 aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Segmentation fault
```
If you don’t observe Segmentation fault, just increase the number of the input as. Here, the message Segmentation fault indicates that your program crashed due to invalid memory access (for instance, refer to memory address 0).

##Exercise 4.

Use gdb, to print the value of the register %eip when the program crashes. How does the program run to this address?
Challenge!

Try to write a C program which prints every return address in the call stack until the invocation of the current function. This is often called a backtrace. This behaves like the bt command in the gdb. Hint: just as the following picture shows, the stack is simply a list with %ebp as the next pointer.
##Note:
Something useful about backtrace
backtrace

You can overwrite the return address with any valid address, instead of eight as (0x61616161). Interestingly, you can supply the starting address of the current buffer being overflowed, whose address has been studied in exercise 1. More interestingly, if the buffer contains some binary code, that code will be executed. Any binary code can be supplied, especially, Alpha One offers a binary code to pop a shell, so this kind of binary code is often called shellcode, although such kind of binary code can do much more interesting things (attacks).
```
$ make test-shell
$ ./test-shell
sh-3.2$ id
uid=1000(seed) gid=1000(seed) groups=4(adm),20(dialout),24(cdrom),
46(plugdev),106(lpadmin),121(admin),122(sambashare),1000(seed)
sh-3.2$ exit
$
```
The -z execstack option will mark the stack segment executable, which you’ll study in detail in lab 2.

May you have seen there is a function name badman in the stack1.c, but we never use it. Now let’s do a exercise, we’ll invoke it manually.

##Exercise 5.
```
$ make stack1
$ gdb stack1 -q
(gdb) b func
Breakpoint 1 at 0x804844a: file stack1.c, line 20.
(gdb) r
Breakpoint 1, func (str=0x8048551 "hello\n") at stack1.c:20
20      strcpy(buffer, str);
(gdb) p badman
$1 = {void ()} 0x804842b <badman>
(gdb) i r $ebp
ebp            0xffffd828    0xffffd828
(gdb) x/wx $ebp+4
0xffffd82c:    0x08048496
(gdb) bt
#0  func (str=0x8048551 "hello\n") at stack1.c:20
#1  0x08048496 in main (argc=1, argv=0xffffd904) at stack1.c:33
(gdb) set *0xffffd82c=0x0804842b
(gdb) c
Continuing.
I am the bad man

Program received signal SIGSEGV, Segmentation fault.
0x08048556 in ?? ()
(gdb)
```
What is stored in the address **0xFFFFd82c** ?  Why after change its value to 0x0804842B, the function *banman* get invoked ? Why the Program result s SegmentFault? IF we don't want to get a SegmentFault, what we should do ?
Exercise 6.
The shellcode we offered can pop up a shell, Now it’s your turn to attack the C program named stack.c using shellcode, you will get a shell if you succeed. You should compile and run your program as follows:
```
$ make stack2
$ ./stack2
sh-3.2$ id
uid=1000(seed) gid=1000(seed) groups=4(adm),20(dialout),24(cdrom),
46(plugdev),106(lpadmin),121(admin),122(sambashare),1000(seed)
sh-3.2$ exit
$
```
##Note:
Here, the -fno-stack-protector option will disable gcc’s stack canary. Hint: you can use the gdb when necessary, but keep in mind that there are some minor differences between the result from gdb and that from the stand-alone executable.
You may look up the Intel Manual to know why the shellcode works.

##Challenge!

Write other kind of (more interesting) shellcode, do whatever you want to do.
#Part B: Buffer Overflows in the Touchstone Web Server

In this part of the lab, you will explore how buffer overflows happen in real-world and how to exploit them. To make the discussion concrete and realistic, you will study a small web server called Touchstone. The touchstone web server is realistic enough to serve static pages (though you can extend it with other features), and meanwhile small enough whose source code can be studied very quickly. We have left some bugs and vulnerabilities in the touchstone web server, some of which will be studied in this lab, whereas others will be studied in future labs.

The Touchstone Web Server

All the source code for touchstone is stored in the code repository. Now compile the touchstone web server and deploy it:
```
$ make
$ ./touchstone
```
##Note:
Open your browser to input this URL http://127.0.0.1:8080, you will get a simple “hello, world” page. If that fails, try to re-run the web server like as ./touchstone 8899, and your browser URL should input http://127.0.0.1:8899. Contact us, if you still have problems.

##Exercise 7.

Study the web server's code, and look for code vulnerability which can be exploited to crash the server by buffer overflows, pay special attention to the file parse.c. Write down a description of each vulnerability in the file named bugs.txt.
Note:
For each vulnerability, how you would construct the input (i.e., the HTTP request) to overflow the buffer, Locate at least one vulnerabilities. Here is a tutorial of the HTTP protocol, you can focus on the GET request.

###Attack the Web Server

Even though the vulnerability has been detected in the web server, it’s still difficult for you to crash the server, because your browser will do most of the dirty work for you that you can not control, that is, you can only do good things with such a browser. So, as a hacker, you have to write your own browser from scratch. We have offered you a simple browser in the file browser.c, basically, this browser will construct an http request and then send to the web server, waiting for the server’s response. The browser is simple enough that is does not come with an html render, however, it’s not difficult to add an html render engine and a UI to make it more realistic. In fact, there are many open source html render engine, say webkit. You are encouraged to do so, if you’re interested in.

##Exercise 8.

For the buffer overflow vulnerability you've found, construct an input to send to the touchstone web server, your goal is to crash the web server (the http server daemon). Note: if you're successful to crash the web server, your browser will remain dead-waiting to receive data from the server. Don't forget that any valid request must end up with \r\n\r\n.
Crashing the web server is just the first step, now you should try to do some thing interesting, say, to delete some sensitive files (for example, the grades.txt). To start with, you can use the program create-shellcode.c, to construct your shellcode, you may have to modify the file according to your need. And you can copy your shellcode to the C program test-shell.c.

##Exercise 9.

Perform your attack by constructing an exploit that hijacks control flow of the web server and unlink (delete) grades.txt. Remember that the web server is on your computer, so you should create a file named grades.txt first.
##Challenge!

Write a remote shellcode, so that you can gain control of a remote machine.
Remote shellcode is used when an attacker wants to target a vulnerable process running on another machine on a local network or intranet.
If successfully executed, the shellcode can provide the attacker access to the target machine across the network.
Remote shellcodes normally use standard TCP/IP socket connections to allow the attacker access to the shell on the target machine.
Such shellcode can be categorised based on how this connection is set up: if the shellcode can establish this connection, it is called a "reverse shell" or a connect-back shellcode because the shellcode connects back to the attacker's machine.
To bypass the firewalls, you can use the port reuse techniques.
Note:
something about remote shellcode and shell code reuse

#Part C: Fixing buffer overflow

The source of buffer overflow vulnerability comes from the web server’s source code, so you should realize the importance to write secure code from the first place, though it’s, nevertheless to say, not easy. For the specific buffer overflows in this lab, you can fix buffer overflows relatively easily by modifying the source code. If you can not gain access to the source code, say your Windows has a buffer overflow (that’s often the case), you will have to wait for M$ to publish a security update.

##Exercise 10.

Try to fix the buffer overflow vulnerabilities of the touchstone web server.
You can use whatever techniques to achieve this, say use safe string copying function strncpy or to allocate the buffer in the heap but not on the stack.
And re-do the attack, observe whether or not your attack will succeed.
#Part D: Hand-in

###Congratulations！
This completes the lab. Remember to hand in your solution to the information system.
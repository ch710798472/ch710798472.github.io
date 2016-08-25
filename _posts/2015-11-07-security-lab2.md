---
layout: post
title: 信息安全（return-to-libc）
category: security
modified: 2015-11-07
tags: [security,return to libc]
comments: true
pinned: true
excerpt: "In this lab, you’ll explore how to defeat the protection mechanisms introduced to counter buffer overflows. The first type of countermeasure is non-executable stack, which will mark the stack memory segment, along with other segments, non-executable..."
---

#Lab 2：Return to Libc

###Lab Overview

　　In this lab, you’ll explore how to defeat the protection mechanisms introduced to counter buffer overflows. The first type of countermeasure is non-executable stack, which will mark the stack memory segment, along with other segments, non-executable. Thus, even the shellcode jumps back to the stack, it has no chance to execute. To defeat this protection mechanism, you will study and use a specific technique called return-to-libc, a special and easy form of return-oriented programming. By using return-to-libc, shellcode can jump to any library code (or any executable code).
　　The second type of countermeasure is memory address layout randomization (ASLR). The key idea of ASLR is to set random addresses for specific memory segments of a given process. Thus, it will be very difficult for an adversary to know the exact address of a buffer. To defeat this protection mechanism, the adversay can guess the starting address of a buffer and attack the server brute-forcely. This is rather realistic, because the range of the stack is relatively small.

　　The third type of countermeasure is a canary. In its simplest form, a canary is an integer on the stack (after the buffer), by checking whether or not a canary is altered, one can check whether or not the buffer is over flowed, just before the function returns. To defeat this, an adversary can just guess the value of the canary. At a first glance, this will be very hard, if not impossible, because the value space is dramatically huge: 232. However, you will understand that under some circumstance, say a web server like Touchstone, it will be very practical. An adversary only need to guess just 1024 times to succeed.

　　This lab consists of three parts:

　　Part A: you will defeat the Non-executable stack protection, by using return-to-lic attack; and
　　Part B: you will defeat the Touchstone web server in a realistic environment: the ASLR is enabled.
　　Lab Environment

　　Download this code to start with. In the first two parts of this lab, you will first disable ASLR:

```
$ su
# sysctl -w kernel.randomize_va_space=0
```
---

##Part A：Non-executable Stack and Return-to-libc Attack

　　To counter buffer overflows, many modern operating systems introduce non-executable stack by marking the stack memory segments non-executable. On old OS, this is achieved by setting up the CS segment register; and modern architecture introduce the NX bit in the page table to help OS manage execution permission. In this part of the lab, you will explore how this protection mechanism can be circumvented.

##Exercise 1.
```
The Ubuntu 12.04 OS you've been using in this lab has the non-executable stack support by default. To compile a C program, just use the -z noexecstack option to mark the stack segment non-executable. Re-compile the vulnerable program stack2.c from lab 1:
$ make stack2
perform a buffer-overflow attack as you do in Lab1, can you succeed any more? What do you observe?
```
The key observation to exploiting buffer overflows with a non-executable stack is that even though you cannot jump to the address of the overflowed buffer (it will not be executable), there’s usually enough code already in the vulnerable program’s address space to perform the operation you want.

　　Thus, to defeat a non-executable stack, you need to first find the (binary) code you want to execute. The most straightforward way is often to find out a function in the standard library, say libc. There are many useful functions in these library, such as execl, system, or unlink, etc.. Then, you need to arrange for the stack to look like a call to that function with the desired arguments, such as system(“/bin/sh”). Finally, you need to arrange for the RET instruction to jump to the function you found in the first step. This attack is often called a return-to-libc attack.

　　Understand the Stack

　　To know how to conduct the return-to-libc attack, it is essential to understand how the stack works. We use a small C program to understand the effects of a function invocation on the stack.

```
/*foobar.c*/
include <stdio.h>
void foo(int x)
{
  printf("Hello world: %d\n", x);
}

int main()
{
  foo(1);
  return 0;
}

```

　　We can use “gcc -S foobar.c” to compile this program to the assembly code. The resulting file foobar.s will look like the following:

```
  8 foo:
  9         pushl   %ebp
 10         movl    %esp, %ebp
 11         subl    $8, %esp
 12         movl    8(%ebp), %eax
 13         movl    %eax, 4(%esp)
 14         movl    $.LC0, (%esp)  : string "Hello world: %d\n"
 15         call    printf
 16         leave
 17         ret

 21 main:
 22         leal    4(%esp), %ecx
 23         andl    $-16, %esp
 24         pushl   -4(%ecx)
 25         pushl   %ebp
 26         movl    %esp, %ebp
 27         pushl   %ecx
 28         subl    $4, %esp
 29         movl    $1, (%esp)
 30         call    foo
 31         movl    $0, %eax
 32         addl    $4, %esp
 33         popl    %ecx
 34         popl    %ebp
 35         leal    -4(%ecx), %esp
 36         ret
```

###Calling and Entering foo()

　　Let us concentrate on the stack while calling foo(). We can ignore the stack before that. Please note that line numbers instead of instruction addresses are used in this explanation.
![](http://i.imgur.com/LsBE0Jg.png)
　　1. Line 28-29: These two statements push the value 1, i.e. the argument to the foo() , into the stack. This operation increments %esp by four. The stack after these two statements is depicted in figure(a).

　　2. Line 30: call foo The statement pushes the address of the next instruction that immediately follows the call statement into the stack (i.e the return address), and then jumps to the code of foo(). The current stack is depicted in figure (b).

　　3. Line 9-10: The first line of the function foo() pushes %ebp into the stack, to save the previous frame pointer. The second line lets %ebp point to the current frame. The current stack is depicted in figure (c).

　　4. Line 11: subl $8, %esp The stack pointer is modified to allocate space (8 bytes) for local variables and the two arguments passed to printf. Since there is no local variable in function foo, the 8 bytes are for arguments only. See figure (d)

###Leaving foo()

　　　　Now the control has passed to the function foo(). Let us see what happens to the stack when the function returns.

1. Line 16: leave This instruction implicitly performs two instructions (it was a macro in earlier x86 releases, but was made into an instruction later):

```
  mov  %ebp, %esp
  pop  %ebp
```

The first statement release the stack space allocated for the function; the second statement recover the previous frame pointer. The current stack is depicted in Figure(e).

2. Line 17: ret This instruction simply pops the return address out of the stack, and then jump to the return address. The current stack is depicted in Figure(f).
3. Line 32: addl $4, %espFurther resotre the stack by releasing more memories allocated for foo. As you can clearly see that the stack is now in exactly the same state as it was before entering the function foo (i.e., before line 28).
###Smashing the Stack

　　Read the virtual file /proc/pid/maps on your machine (pid is the process id) you can find and locate the key library such as libc-2.22.so, which contains many useful functions. The key function you may use to attack the target may be system(), unlink() and so on.

---
##Exercise 2.

　　Use gdb to smash the function stack, the C program offered you here is exec3.c. Note that, the function address at your PC may be different from mine, but just take it easy.

```
$ make exec3
$ gdb -q exec3
Reading symbols from exec3...done.
(gdb) b fun
Breakpoint 1 at 0x8048423: file exec3.c, line 6.
(gdb) r
Starting program: /home/seed/sand-box/exec3

Breakpoint 1, fun () at exec3.c:6
        printf("Return to fun!\n");
(gdb) p system
$1 = {} 0xb7e583b0 <__libc_system>
(gdb) p/x $ebp+4
$2 = 0xbffff3cc
(gdb) set *0xbffff3cc=0xb7e583b0
(gdb) p/x $ebp+16
$3 = 0xbffff3d8
(gdb) set *0xbffff3d8=0x736c
(gdb) x/s 0xbffff3d8
0xbffff3d8:    "ls"
(gdb) p/x $ebp+12
$4 = 0xbffff3d4
(gdb) set *0xbffff3d4=0xbffff3d8
(gdb) c
Continuing.
Return to fun!
exec3  exec3.c nop-overflow.c    stack1.c  stack.c  test.c  x

Program received signal SIGSEGV, Segmentation fault.
0xbffff3d8 in ?? ()
```

　　As you can see, the command system(“ls”) constructed by gdb runs smoothly, but not perfect. What triggered the “SIGSEG” fault? Modify the process memory in gdb just likeabove, to to let the process exit gracefully.

　　Ret-to-libc Attack

　　Till now, you already know how the function call stack is organized and how to find the library function address. So you can try to attack the Touchstone web server using ret-to-libc.

---
##Exercise 3.

　　First, compile the Touchstone webserver:

```
  $ cd server
  $ make all
  $ ./touchstone
```

　　Now, try to perform a return-to-libc attack by contructing and sending a malicious request containing your shellcode. Your shellcode can still delete a file from the web server, or can do something else.

　　The Environment Variables

　　Sometime, the buffer that we can smash may not big enough to store a “big” shellcode. But we may find something useful in the Environment Variables of target machine which may help us to perform the attack.

　　Make sure you have read the paper return-to-libc.pdf carefully cause there is something useful in it.
　　Also you can get more information about Environment Variables from the Internet like Wikipedia

##Challenge!

　　Ret-to-libc attack with Environment Variables

```
Perform a ret-to-libc attack with using Environment Variables to do someting like popup a shell.(assume the target only have a "small" buffer)
```

---
##Part B: Address Space Layout Randomization

####ASLR

　　Address space layout randomization is a technique used to fortify systems against buffer overflow attacks. The idea is to introduce artificial diversity by randomizing the memory location of certain system components. This mechanism is available for both Linux (via PaX ASLR) and OpenBSD.

　　Now, turn on the Ubuntu’s address space layout randomization:

```
 $ su root
 #/sbin/sysctl -w kernel.randomize_va_space=2
```

##Exercise 4. 
　　After turning on the ASLR, compile the Touchstone web server:
```
$make clean
$make all
$./touchstone
```
　　Try to attack the webserver using buffer overflow. Can you succeed? Where is the buffer’s address? Is it exploitable?

###Brute-Force Attack

　　　To defeat ASLR, we can use the Brute Force attack technique, which is simple but effective in guessing the variable buffer address. The basic idea is that although we don’t know the exact address of the buffer, however, we know its range, say, from 0x00000000 to 0xbfffffff. So, by trying each address in turn, we’ll hit the right address sooner or later.

---
##Exercise 5.

　　After turning on the ASLR, compile the Touchstone web server:
```
$make clean
$make exec5
$./touchstone
```
　　Send a request to the server containing the guessed address, repeating this process until you succeed. (Hint: think carefully the possible address of the buffer (hence the stack). Is every address between 0x00000000 to 0xbfffffff possible? Is every address in the given stack a candidate for the buffer address? Answering these questions as precisely as possible will speed up your brute-force attack dramatically.)

####SomeThing Interesting

　　The browser you’ve been using so far is very simple: basically, it can only read reponse from the web server (as a sequence of characters). A real-world browser tends to be more complex: besides reads response, it also renders html, run JavaScript engine, perfrom various security checking, etc.. So this means that a real-world browser tends to contain more vulnerabilities. So you can investigate how to attack real-world browsers using the knowledge you’ve learned. You can browse through the known vulnerabilities on CVE, and you can use some tools like metasploit to make the work of creating shellcode simpler.
　　Turn on ASLR, enable non-executable stack, enable stack canary, and perform buffer overflow attack on the Touchstone web server. Do you have any chance to secceed?(The answer is yes.)

---
##Hand-In

　　This completes the lab. Remember to hand in your solution to the information system.

---
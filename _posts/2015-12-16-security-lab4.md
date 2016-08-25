---
layout: post
title: 信息安全（privilege-separation）
category: security
modified: 2015-12-16
tags: [security,setuid]
comments: true
pinned: true
excerpt: "This lab consists of three parts:Part A: you will explore identity forgery, in which a malicious user can act as a victim user;Part B: you will explore the possibility of information disclosure during network transmission;Part C: you will secure the server by designing some cryptography algorithm and adding them to your server...."
---
[题目要求](http://pan.baidu.com/s/1sjSvagt)密码：1ui1
[初始源码](http://pan.baidu.com/s/1bnYTkuz) 密码：4agj

#实验解答如下
##一、实验内容：
	This lab consists of three parts:
    Part A: you will explore identity forgery, in which a malicious user can act as a victim user;
    Part B: you will explore the possibility of information disclosure during network transmission;
    Part C: you will secure the server by designing some cryptography algorithm and adding them to your server.

##二、实验步骤：
###Exercise 1:
    1. 用户可以转账任何多的钱，及时他没有那么多的钱。
    2. 用户可以输入小于零的一个转账数字来增加自己的钱。
    3. 用户可以转账给任何人，及时这个人不存在，这个时候钱就进了黑洞了。
    4. 用户只能转账整数，这是因为设计的时候只能存整数。
    5. 用户可以转账给自己。

###Exercise 2:
　　首先我们需要对转账的数目进行边界检查，然后在对转账的用户进行检查，就可以了。在handle.c文件的handlePostTransfer()函数中添加下列代码

```
    int t = Db_checkUser(to);
    if(!t)//user don't exist{
      printf("user don't exist!");
      return;}
  	if(money <= 0 ){
    	printf("can't transefer money<=0!");
    	return;}
  	if(!strcmp(from,to)){
    	printf("can't transfer money to yourself!");
     	return;}
 	int fromBalance = Db_readBalance (from);
  	if(fromBalance < money){
    printf("you don't hava enough money!");
    return;}

```

运行后再检查上面的bug，发现实际可行，具体如下图：
![](http://i.imgur.com/UffI4dk.jpg)
![](http://i.imgur.com/7MGsFiX.jpg)
![](http://i.imgur.com/dt3vx6J.jpg)

###Exercise 3:
　　这次的转账的功能具体是写在handle.c文件中的，其中对于数据库的操作全都写在了sql-lit3/dbutil.c文件中，每次登陆之后，有httpd进行解析看看接受的是什么类型的信息，然后分发给banksv等，然后具体对于请求的解析是由parse.c文件中的相关函数解析的，由于这次添加了POST的请求，所以相比于上次实验多了如何处理POST请求的部分，解析完毕之后就可以交给handle.c进行主要的功能处理了。

###Exercise 4:
　　我们可以借助firefox自带的firebug来查看POST的请求具体的值。
　　我们可以发现四个参数具体值，其中submit_transfer参数代表了是执行转账操作，下面三个分别是转账人、转账多少、转账给谁。那么我们就可以利用这个来构造一个POST请求，但是这里我们为了方便快捷，可以使用另外一个firefox自带的插件工具Live HTTP headers，利用其中replay功能，进行重放攻击，并且可以任意修改其中的参数，正好适合我们构造POST请求，如下图所示，由aaa转账给bbb，并且很容易从右边的服务器命令行输出的信息判断是成功的：

![](http://i.imgur.com/xcTVJ8K.jpg)
![](http://i.imgur.com/f5F5ZVu.jpg)
![](http://i.imgur.com/1UGObEn.jpg)
 
###Exercise 5:
　　首先我把分成两步，第一步是利用工具构造一个cookie，然后构造post请求，第二步再由服务器返回一个cookie给浏览器。
	1.	还是利用firebug创建一个cookie名字是ch值是1，
![](http://i.imgur.com/odAbFeO.jpg)
	2.	构建一个带cookie的post请求，发现服务器成功解析了cookie，当然了需要在parse.c中parseHeaders函数加入如下代码，具体位置参照图片中位置。
![](http://i.imgur.com/JIe9L7I.png)
![](http://i.imgur.com/krctvxf.jpg)
	3.	再观察到底用户的钱有没有正确的处理，下面两张图对比可得知确实转账成功（注意操作时间和余额）。
![](http://i.imgur.com/c22IEs0.jpg)
![](http://i.imgur.com/lFrRr4i.jpg)
	4.	如果我们构造的请求cookie有错的话会出现下图所示的结果，回到登录界面当然也需要另外再加代码，在handle.c中handlePost函数中添加如下代码：
![](http://i.imgur.com/i9VfizB.png)
　　其中getCookie()代码是取得刚才上一步服务器中解析cookie的值。
![](http://i.imgur.com/X1k6t6Y.jpg)
	5.	利用请求中的Set-Cookie来给浏览器发送一个cookie，这需要在handle.c文件的handlePostLogin函数中添加如下代码：
 ![](http://i.imgur.com/kThMF9F.png)
　　重新编译服务器代码，运行发现浏览器多了一个我们设置的ch=100的cookie，请看cookie存在时间和value的值。
![](http://i.imgur.com/b7UikX4.jpg)
	6.	那么这样一来，浏览器就有来自于服务器的cookie了，重复2-4步骤完全正确。

###Exercise 6:
　　首先利用wirshark监听lo，用来抓虚拟机的包，然后在浏览器进行几次操作以便取得包，再筛选出http请求的包，找到POST request，发现跟利用Live HTTP headers抓到的请求结构差不多，那么很容易就构造一个虚假的POST request，结果跟上面的一样，就不重复放图了。

###Exercise 7:
    1. 首先需要写三个函数实现添加cookie到数据库（Db_readCookie），从数据库中读取cookie（Db_writeCookie）,以及对cookie进行加密（encryp_cookie），这些都放在sql-lite3/dbutil.c文件中，当然得先在user表中添加cookie字段（VARCHAR(50)）：

```
	char* Db_readCookie (const char *name)
	{
    if(DEBUG)
    printf("Db_readCookie: name=[%s]\n", name);
  
    if(Db_open(0)!=SUCCESS){
      fprintf(stderr
	    , "open failed![%s]\n"
	   , sqlite3_errmsg(db));
    return 0;
    }
    char sql[1024];
    sprintf(sql, "SELECT cookie from user WHERE name = '%s'", name);
    int row, column;
    char **result;
    char *errorMsg;
    int ret = sqlite3_get_table(db, sql, &result, &row, &column, &errorMsg);
    if (ret!=SQLITE_OK){
      fprintf (stderr
	     , "error read cookie\n");
    return 0;
    }
    if(DEBUG){
      printf("nrow=[%d], column=[%d]\n", row, column);
      printf ("======%s\n", (result+1)[column-1]);
    }
    char *cookies = (result+1)[column-1];
    sqlite3_free_table(result);
    //Db_exec(db, sql);
    sqlite3_close(db);
    return cookies;
    }

    int Db_writeCookie (const char *name
		     , const char *cookie)
    {
    if(DEBUG)
      printf("Db_writecookie(): name=[%s]\n"
	   , name);
  
    if(Db_open(0)!=SUCCESS){
      fprintf(stderr
	    , "open failed![%s]\n"
	   , sqlite3_errmsg(db));
    return 0;
    }
    char sql[1024];
    char *errorMsg;
    //encryp
    char copy_cookie[50];
    strcpy(copy_cookie,cookie);
    encryp_cookie(copy_cookie);
    sprintf(sql, "UPDATE user set cookie=%s where name= '%s'", copy_cookie, name);
    int ret = sqlite3_exec(db, sql, 0, 0, &errorMsg);
    if (ret!=SQLITE_OK){
      fprintf (stderr
	     , "%s\n"
	     , "error read cookie");
    return 0;
    }
    if(DEBUG){
      printf("Db_writeCookie() succeed\n");
    }
    sqlite3_close(db);
    return 1;
    }
    //encryption function，key =3
    void encryp_cookie(char *cookie)
    {
      int i = 0;
      char temp;
      while(cookie[i] != '\0')
      {
        temp = cookie[i]^3;
        cookie[i] = temp;
        i++;
      } 
     }

```

　　2. 接下来需要在handle.c中handlePost函数里实现解密的部分，其实解密就是用秘钥数字3来进行异或运算即可：

```
    //add to check cookie
    char *cookie = getCookie();//cookie: aaa=222
    int c_t = 0;
	//depart username from cookie
    while(cookie[c_t] != '=')
    {
       c_t++;
    }
    
    //decryp cookie
    char temp;
    c_t++;
    int c_i = c_t;
    while(cookie[c_t] != '\0')
    {
       temp = cookie[c_t] ^ 3;
       cookie[c_t] = temp;
       c_t++;
    }
    printf("decryp cookie =%s\n",cookie);
    if(strcmp(&cookie[c_i],"111")==0) //just compare cookie not include username
      handlePostTransfer (fd, from, to, m);
    else
    {
      printf("fault cookie\n");
      handlePostLogout(fd);//faile to load index.html
    }

```

　　4. 我用的cookie值是111，用3进行异或加密之后是222，解密之后也应该是111，结果完全符合，如下图所示：
![](http://i.imgur.com/CujOUuy.jpg)
![](http://i.imgur.com/HdwynJ8.jpg)
 　　5. 实验结果，请注意时间的对比，防伪。
![](http://i.imgur.com/PzeLNC8.png)

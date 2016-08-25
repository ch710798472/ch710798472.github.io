---
layout: post
title: 信息安全（privilege-separation）
category: security
modified: 2015-11-26
tags: [security,setuid]
comments: true
pinned: true
excerpt: "In this lab, you’ll explore privilege separation. The key insight of privilege separation is　to give minimal privilege to each component of a system, so that when one component　of the system is comprised, other components will not be comprised too..."
---

#Lab 3:privilege-separation　　

####　　In this lab, you’ll explore privilege separation. The key insight of privilege separation is　to give minimal privilege to each component of a system, so that when one component　of the system is comprised, other components will not be comprised too.　　　

####　　To make the discussion concrete, you will do this lab for the Touchstone web server, that is, you will privilege-separate the Touchstone web server by giving each component appropriate privilege. 　　

###　　To be specific, you will first examine possible bugs in the source code of the Touchstone web server, and comprise the Touchstone web server by designing and performing exploitations. Finally, you will break up the application into privilege-separated components to minimize the effects of possible vulnerabilities.　　

###This lab consists of three parts:　　

###Part A: you will examine the architecture of the Touchstone web server. The Touchstone web server in this lab differs dramatically from those from lab 1 and 2, the current one is based on the idea of services;
###Part B: you will explore jail, by which you can constraint the service in some fake root directory; and
###Part C: you will privilege-separate the Touchstone web server by assigning each component appropriate privilege.

###　　I got really really so busy, which let me upload all the files. More details plase download files from [baidu yunpan](http://pan.baidu.com/s/1nts50oT) passwd: a28p

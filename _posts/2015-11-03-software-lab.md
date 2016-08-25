---
layout: post
title: 软件工程--设计一个可重用基于链表的菜单程序
category: software
modified: 2015-11-03
tags: [software,ustc]
comments: true
pinned: true
excerpt: " 1. 用可重用的链表模块来实现命令行菜单小程序，执行某个命令时调用一个特定的函数作为执行动作；2. 链表模块的接口设计要足够通用，命令行菜单小程序的功能保持不变；3. 可以将通用的Linktable模块集成到我们的menu程序中；4. 接口规范，使用callback函数；..."
---

#内容  
1. 用可重用的链表模块来实现命令行菜单小程序，执行某个命令时调用一个特定的函数作为执行动作；
2. 链表模块的接口设计要足够通用，命令行菜单小程序的功能保持不变；
3. 可以将通用的Linktable模块集成到我们的menu程序中；
4. 接口规范，使用callback函数；
5. 为menu子系统设计接口，并写用户范例代码来实现原来的功能
6. 使用make和make clean来编译程序和清理自动生成的文件
7. 使menu子系统支持带参数的复杂命令，自定义一个带参数的复杂命令get()函数（get -n name -v version命令）
8. 使用getopt函数获取命令行参数  

#代码  
##Makefile  
```
CC_PTHREAD_FLAGS	=	-lpthread
CC_FLAGS	        =	-c
CC_OUTPUT_FLAGS     =	-o
CC	              =	gcc
RM	              =	rm
RM_FLAGS	        =	-f
TARGET	          = test
OBJS            	=	linktable.o menu.o test.o
all:	$(OBJS)
$(CC) $(CC_OUTPUT_FLAGS) $(TARGET) $(OBJS)
.c.o:
$(CC) $(CC_FLAGS) $<
clean:
$(RM) $(RM_FLAGS) $(OBJS) $(TARGET) *.bak  
```
##menu.c
```
/**************************************************************************************************/
/* Copyright (C) SSE@USTC, 2015                                                                   */
/*                                                                                                */
/*  FILE NAME             :  menu.c                                                               */
/*  PRINCIPAL AUTHOR      :  ch                                                                   */
/*  SUBSYSTEM NAME        :  menu                                                                 */
/*  MODULE NAME           :  menu                                                                 */
/*  LANGUAGE              :  C                                                                    */
/*  TARGET ENVIRONMENT    :  ANY                                                                  */
/*  DATE OF FIRST RELEASE :  2015/10/30                                                           */
/*  DESCRIPTION           :  This is a menu program                                               */
/**************************************************************************************************/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "menu.h"
#include "linktable.h"
int Help();
#define CMD_MAX_LEN 128
#define DESC_LEN    1024
#define CMD_NUM     10
#define CMD_MAX_ARGV_NUM 10
typedef struct DataNode
{
    tLinkTableNode * pNext;
    char*   cmd;
    char*   desc;
    int     (*handler)(int argc, char* argv[]);
} tDataNode;
int SearchCondition(tLinkTableNode * pLinkTableNode, void *args)
{
	char *cmd = (char *)args;
    tDataNode * pNode = (tDataNode *)pLinkTableNode;
    if(strcmp(pNode->cmd, cmd) == 0)
    {
        return  SUCCESS;  
    }
    return FAILURE;	       
}
/* find a cmd in the linklist and return the datanode pointer */
tDataNode* FindCmd(tLinkTable * head, char * cmd)
{
    return  (tDataNode*)SearchLinkTableNode(head,SearchCondition,(void *)cmd);
}
/* show all cmd in listlist */
int ShowAllCmd(tLinkTable * head)
{
    tDataNode * pNode = (tDataNode*)GetLinkTableHead(head);
    while(pNode != NULL)
    {
        printf("%s - %s\n", pNode->cmd, pNode->desc);
        pNode = (tDataNode*)GetNextLinkTableNode(head,(tLinkTableNode *)pNode);
    }
    return 0;
}
/* menu program */
tLinkTable * head = NULL;
   
/** 
 * add command to menu; 
 * initiate commandline;
 */
int MenuConfig(char *cmd, char *desc, int (*handler)(int argc, char *argv[]))
{
	tDataNode *pNode = NULL;
	if (head == NULL)
	{
		head = CreateLinkTable();
		pNode = (tDataNode*)malloc(sizeof(tDataNode));
		pNode->cmd = "help";
		pNode->desc = "Menu List";
		pNode->handler = Help;
		AddLinkTableNode(head, (tLinkTableNode *)pNode);
	}
	pNode = (tDataNode*)malloc(sizeof(tDataNode));
	pNode->cmd = cmd;
	pNode->desc = desc;
	pNode->handler = handler;
	AddLinkTableNode(head, (tLinkTableNode *)pNode);
	return 0;
}
/* Menu Engine Execute */
int ExecuteMenu()
{
	int argc = 0;
	char cmd_temp[CMD_MAX_LEN];
   	char *pcmd = NULL;
	char *argv_temp[CMD_MAX_ARGV_NUM];
	char *cmd = NULL;
	char **argv = NULL;
	char **argv_ini = argv_temp;
	/* cmd line begins */
    while(1)
    {
		// initiate before circle
		cmd = cmd_temp;
		argv = argv_ini;
		argc = 0;
		pcmd = NULL;
        printf("Input a cmd number > ");
        //scanf("%s", cmd);
		pcmd = fgets(cmd, CMD_MAX_LEN, stdin);
		/* input Nothing */
		if(pcmd == NULL)
		{
			continue;
		}
		/* convert pcmd to argc/argv[] */
		pcmd = strtok(pcmd, " ");
		while (pcmd != NULL && argc < CMD_MAX_ARGV_NUM)
		{
			argv[argc] = pcmd;
			argc++;
			pcmd = strtok(NULL, " ");
		}
		if (argc == 1)
		{
			int length = strlen(argv[0]);
			/* change '\n' to '\0' at the end of the cmd */
			*(argv[0] + length - 1) = '\0';
		}
		//printf("Argv[0] is %s\n", argv[0]);
        
		tDataNode *p = FindCmd(head, argv[0]);
        if( p == NULL)
        {
            printf("This is a wrong cmd!\n ");
            continue;
        }
        printf("%s - %s\n", p->cmd, p->desc); 
        if(p->handler != NULL) 
        {
            p->handler(argc, argv);
        }
	} 
}
int Help()
{
    ShowAllCmd(head);
    return 0; 
}
```  
##menu.h
```
#ifndef _MENU_H
#define _MENU_H
/* add command to menu */
int MenuConfig(char *cmd, char *desc, int (*handler)(int argc, char* argv[]));
/* Menu Engine Execute */
int ExecuteMenu();
#endif
```  
##linktable.c
```
/********************************************************************/
/* Copyright (C) SSE-USTC, 2015                                     */
/*                                                                  */
/*  FILE NAME             :  linktabe.c                             */
/*  PRINCIPAL AUTHOR      :  ch                                     */
/*  SUBSYSTEM NAME        :  LinkTable                              */
/*  MODULE NAME           :  LinkTable                              */
/*  LANGUAGE              :  C                                      */
/*  TARGET ENVIRONMENT    :  ANY                                    */
/*  DATE OF FIRST RELEASE :  2015/10/30                             */
/*  DESCRIPTION           :  interface of Link Table                */
/********************************************************************/
#include<stdio.h>
#include<stdlib.h>
#include"linktable.h"
struct LinkTable
{
	tLinkTableNode *pHead;
	tLinkTableNode *pTail;
	int			SumOfNode;
	pthread_mutex_t mutex;
};
/*
 * Create a LinkTable
 */
tLinkTable * CreateLinkTable()
{
    tLinkTable * pLinkTable = (tLinkTable *)malloc(sizeof(tLinkTable));
    if(pLinkTable == NULL)
    {
        return NULL;
    }
    pLinkTable->pHead = NULL;
    pLinkTable->pTail = NULL;
    pLinkTable->SumOfNode = 0;
    pthread_mutex_init(&(pLinkTable->mutex), NULL);
    return pLinkTable;
}
/*
 * Delete a LinkTable
 */
int DeleteLinkTable(tLinkTable *pLinkTable)
{
    if(pLinkTable == NULL)
    {
        return FAILURE;
    }
    while(pLinkTable->pHead != NULL)
    {
        tLinkTableNode * p = pLinkTable->pHead;
        pthread_mutex_lock(&(pLinkTable->mutex));
        pLinkTable->pHead = pLinkTable->pHead->pNext;
        pLinkTable->SumOfNode -= 1 ;
        pthread_mutex_unlock(&(pLinkTable->mutex));
        free(p);
    }
    pLinkTable->pHead = NULL;
    pLinkTable->pTail = NULL;
    pLinkTable->SumOfNode = 0;
    pthread_mutex_destroy(&(pLinkTable->mutex));
    free(pLinkTable);
    return SUCCESS;		
}
/*
 * Add a LinkTableNode to LinkTable
 */
int AddLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode)
{
    if(pLinkTable == NULL || pNode == NULL)
    {
        return FAILURE;
    }
    pNode->pNext = NULL;
    pthread_mutex_lock(&(pLinkTable->mutex));
    if(pLinkTable->pHead == NULL)
    {
        pLinkTable->pHead = pNode;
    }
    if(pLinkTable->pTail == NULL)
    {
        pLinkTable->pTail = pNode;
    }
    else
    {
        pLinkTable->pTail->pNext = pNode;
        pLinkTable->pTail = pNode;
    }
    pLinkTable->SumOfNode += 1 ;
    pthread_mutex_unlock(&(pLinkTable->mutex));
    return SUCCESS;		
}
/*
 * Delete a LinkTableNode from LinkTable
 */
int DelLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode)
{
    if(pLinkTable == NULL || pNode == NULL)
    {
        return FAILURE;
    }
    pthread_mutex_lock(&(pLinkTable->mutex));
    if(pLinkTable->pHead == pNode)
    {
        pLinkTable->pHead = pLinkTable->pHead->pNext;
        pLinkTable->SumOfNode -= 1 ;
        if(pLinkTable->SumOfNode == 0)
        {
            pLinkTable->pTail = NULL;	
        }
        pthread_mutex_unlock(&(pLinkTable->mutex));
        return SUCCESS;
    }
    tLinkTableNode * pTempNode = pLinkTable->pHead;
    while(pTempNode != NULL)
    {    
        if(pTempNode->pNext == pNode)
        {
            pTempNode->pNext = pTempNode->pNext->pNext;
            pLinkTable->SumOfNode -= 1 ;
            if(pLinkTable->SumOfNode == 0)
            {
                pLinkTable->pTail = NULL;	
            }
            pthread_mutex_unlock(&(pLinkTable->mutex));
            return SUCCESS;				    
        }
        pTempNode = pTempNode->pNext;
    }
    pthread_mutex_unlock(&(pLinkTable->mutex));
    return FAILURE;		
}
/*
 * Search a LinkTableNode from LinkTable
 * int Conditon(tLinkTableNode * pNode);
 */
tLinkTableNode * SearchLinkTableNode(tLinkTable *pLinkTable, int Conditon(tLinkTableNode * pNode, void *args), void *args)
{
    if(pLinkTable == NULL || Conditon == NULL)
    {
        return NULL;
    }
    tLinkTableNode * pNode = pLinkTable->pHead;
    while(pNode != NULL)
    {    
        if(Conditon(pNode, args) == SUCCESS)
        {
            return pNode;				    
        }
        pNode = pNode->pNext;
    }
    return NULL;
}
/*
 * get LinkTableHead
 */
tLinkTableNode * GetLinkTableHead(tLinkTable *pLinkTable)
{
    if(pLinkTable == NULL)
    {
        return NULL;
    }    
    return pLinkTable->pHead;
}
/*
 * get next LinkTableNode
 */
tLinkTableNode * GetNextLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode)
{
    if(pLinkTable == NULL || pNode == NULL)
    {
        return NULL;
    }
    tLinkTableNode * pTempNode = pLinkTable->pHead;
    while(pTempNode != NULL)
    {    
        if(pTempNode == pNode)
        {
            return pTempNode->pNext;				    
        }
        pTempNode = pTempNode->pNext;
    }
    return NULL;
}
```
##linktable.h
```
/********************************************************************/
/* Copyright (C) SSE-USTC, 2015                                     */
/*                                                                  */
/*  FILE NAME             :  linktabe.h                             */
/*  PRINCIPAL AUTHOR      :  ch                                     */
/*  SUBSYSTEM NAME        :  LinkTable                              */
/*  MODULE NAME           :  LinkTable                              */
/*  LANGUAGE              :  C                                      */
/*  TARGET ENVIRONMENT    :  ANY                                    */
/*  DATE OF FIRST RELEASE :  2015/10/30                             */
/*  DESCRIPTION           :  interface of Link Table                */
/********************************************************************/
#ifndef _LINK_TABLE_H_
#define _LINK_TABLE_H_
#include <pthread.h>
#define SUCCESS 0
#define FAILURE (-1)
/*
 * LinkTable Node Type
 */
typedef struct LinkTableNode
{
    struct LinkTableNode * pNext;
}tLinkTableNode;
/*
 * LinkTable Type
 */
typedef struct LinkTable tLinkTable;
//typedef struct LinkTable
//{
//    tLinkTableNode *pHead;
//    tLinkTableNode *pTail;
//    int			SumOfNode;
//    pthread_mutex_t mutex;
//}tLinkTable;
/*
 * Create a LinkTable
 */
tLinkTable * CreateLinkTable();
/*
 * Delete a LinkTable
 */
int DeleteLinkTable(tLinkTable *pLinkTable);
/*
 * Add a LinkTableNode to LinkTable
 */
int AddLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode);
/*
 * Delete a LinkTableNode from LinkTable
 */
int DelLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode);
/*
 * Search a LinkTableNode from LinkTable
 * int Conditon(tLinkTableNode * pNode);
 */
tLinkTableNode * SearchLinkTableNode(tLinkTable *pLinkTable, int Conditon(tLinkTableNode * pNode, void *args), void *args);
/*
 * get LinkTableHead
 */
tLinkTableNode * GetLinkTableHead(tLinkTable *pLinkTable);
/*
 * get next LinkTableNode
 */
tLinkTableNode * GetNextLinkTableNode(tLinkTable *pLinkTable,tLinkTableNode * pNode);
#endif /* _LINK_TABLE_H_ */
```
##test.c
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include "menu.h"
extern char *optarg;
extern int cmderror;
extern int optind;
int Quit(int argc, char *argv[])
{
	exit(0);
}
int Get(int argc, char *argv[])
{
	if (argc == 1)
	{
		printf("get cmd is none!\n");
		return 0;
	}
	int cmd = 's';
	
	while((cmd = getopt(argc, argv, "n:v:")) != -1)
	{
		switch(cmd)
		{
			case 'n':
				printf("Get, argv is %s\n", optarg); break;
			case 'v':
				printf("Get, argv is %s\n", optarg); break;
			case ':':
				switch (cmd)
				{
					case 'n':
						printf("Get, -n:\n"); break;
					case 'v':
						printf("Get, -v:\n"); break;
					case '\n':
						break;
					default:
						printf("Usage: %s [-n] [-v] argc\n", argv[0]);
				}
				break;
			default:
				printf("Usage: %s [-n argc1] [-v argc2] \n", argv[0]); break;
		}
	}
	optind = 1;
	return 0;
}
int main(int argc, char *argv[])
{
	MenuConfig("version", "Menu program v2.0!", NULL);
	MenuConfig("quit", "Quit from Menu", Quit);
	MenuConfig("get", "Get something", Get);
	ExecuteMenu();
}
```
##testlintable.c
```
/**************************************************************************************************/
/* Copyright (C) SSE@USTC, 2015                                                                   */
/*                                                                                                */
/*  FILE NAME             :  testlinktable.c                                                               */
/*  PRINCIPAL AUTHOR      :  ch                                                                   */
/*  SUBSYSTEM NAME        :  testlinktable                                                                 */
/*  MODULE NAME           :  testlinktable                                                                 */
/*  LANGUAGE              :  C                                                                    */
/*  TARGET ENVIRONMENT    :  ANY                                                                  */
/*  DATE OF FIRST RELEASE :  2015/10/30                                                           */
/*  DESCRIPTION           :  This is a testlinktable program                                               */
/**************************************************************************************************/
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
#include"linktable.h"
#define debug   
typedef struct Node
{
    tLinkTableNode * pNext;
    int data;
}tNode;
tNode * Search(tLinkTable *pLinkTable);
int SearchCondition(tLinkTableNode * pLinkTableNode);
int main()
{
    int i;
    tLinkTable * pLinkTable = CreateLinkTable();
    assert(pLinkTable != NULL);
    for(i = 0; i < 10; i++)
    {
        tNode* pNode = (tNode*)malloc(sizeof(tNode));
        pNode->data = i;
        debug("AddLinkTableNode\n");
        AddLinkTableNode(pLinkTable,(tLinkTableNode *)pNode);
    }
    /* search by callback */
    debug("SearchLinkTableNode\n");
    tNode* pTempNode = (tNode*)SearchLinkTableNode(pLinkTable,SearchCondition);
    printf("%d\n",pTempNode->data);
    /* search one by one */
    pTempNode = Search(pLinkTable);
    printf("%d\n",pTempNode->data);
    debug("DelLinkTableNode\n");
    DelLinkTableNode(pLinkTable,(tLinkTableNode *)pTempNode);
    free(pTempNode);
    DeleteLinkTable(pLinkTable);
}
tNode * Search(tLinkTable *pLinkTable)
{
    debug("Search GetLinkTableHead\n");
    tNode * pNode = (tNode*)GetLinkTableHead(pLinkTable);
    while(pNode != NULL)
    {
        if(pNode->data == 5)
        {
            return  pNode;  
        }
        debug("GetNextLinkTableNode\n");
        pNode = (tNode*)GetNextLinkTableNode(pLinkTable,(tLinkTableNode *)pNode);
    }
    return NULL;
}
int SearchCondition(tLinkTableNode * pLinkTableNode)
{
    tNode * pNode = (tNode *)pLinkTableNode;
    if(pNode->data == 6)
    {
        return  SUCCESS;  
    }
    return FAILURE;	       
}
```
![](http://i.imgur.com/HregQ3e.png)
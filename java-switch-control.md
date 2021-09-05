---
title: java学习笔记2-流程控制 Demo
name: java-switch-control
date: 2018-07-17
id: 0
tags: [java,学习笔记]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


    /*
    利用for循环实现9*9的乘法表
    */


​    
```java
public class for99
{
	public static void main(String []args)
	{ for (int i=1;i<10;i++)
		{ for (int j=1;j<10;j++)
			{ if (j<=i)
				{ System.out.print(i+"*"+j+"="+(i*j)+" ");
				}
			}
		System.out.println(" ");
		}
	}
}
```

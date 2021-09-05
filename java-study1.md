---
title: java学习笔记1
name: java-study1
date: 2018-07-07
id: 0
tags: [java,学习笔记]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


定义的java程序的时候的类名字必须和文件的名称保持一致否则后面会出现编译的错误

在第一次的hello world编译中书写的代码如下

```java
//2018.7.7
//无敌的皮皮
//java学习的第一个程序helloworld
public class Hello
{  //程序的入口
    public static void main(String args[])
    {
        System.out.println("hello world");
    }
}
```

 


第一次我使用的是notapad++的编译器，然而默认的是UTF8的编码字符，在使用命令javac Hello.java的时候出现了编译的错误

因为教学使用的是记事本，记事本默认的编码是ansi，在改变了编码后，把错误的编码删除后再次利用命令行编译程序后成功编译

![PeUDxg.png](https://s1.ax1x.com/2018/07/07/PeUDxg.md.png)

因为教学使用的是记事本，记事本默认的编码是ansi，在改变了编码后，把错误的编码删除后再次利用命令行编译程序后成功编译
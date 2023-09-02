---
title: java学习笔记6-封装
name: java-study6
date: 2018-07-26
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


封装是把抽象的数据和对数据的操作封装在一起，这样数据就被保护了起来，只有得到授权的操作（类方法）才能去访问

包的必要性：什么是包？

com.xiaoming中.代表分层

当定义的类的名字相同时用包来区分

*打包命令一般放在文件开始的地方

*包的命名规范，全部小写

引入包：import

例如引入HashMap包，import java.util.*;

用包来控制访问的级别

```java
package com.xiaoqiang;
//打包命令，把生成的字节码放在该包下

public class xiaoqiang {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
    Dog dog1=new Dog();
    
	}

}

class Dog
{
	public int a;
	protected String name;
	String color;
	private float price;
	
	public void main()
	{
		System.out.println(a);
	}
}
```
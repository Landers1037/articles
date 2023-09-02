---
title: java学习笔记8-多态
name: java-study8
date: 2018-07-26
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


一个引用在不同情况下的多种状态

java允许父类的引用变量引用子类的实例对象

Animal a=new Cat();

这种转换是自动完成的

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
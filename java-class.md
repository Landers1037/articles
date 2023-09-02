---
title: 抽象类
name: java-class
date: 2018-07-28
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


```java
package lrj_demo;
/*
 * 2018.7.28
 * 抽象类 
 */

public class demo1{
	public static void main(String []args){
		System.out.println("nobody");
		//Master m=new slave();
		//m.call();
}
}

abstract class Master{
	//i am nobody
	//我是个什么？
	//抽象类可以没有抽象方法
	//抽象类的不确定性，对方法不能实例化，也就是后面不能加{}
	abstract public void  call();
}

//子类的继承
//java要求我们把父类里的所有的抽象方法都实现
class slave extends Master {
	//对父类的方法的重载
	//我是一个锤子
	public  void call(){
		System.out.println("i am a hammer");
	}
}
```
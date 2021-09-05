---
title: java学习笔记5-类方法
name: java-study5
date: 2018-07-26
id: 0
tags: [java]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


类方法（又为静态方法）

//java里的规则类变量原则上用类方法去访问

//类方法中不能访问非类变量，比如静态的方法只能访问静态的变量，普通方法却可以访问其他的变量

```java
/*
2018.7.25
静态变量的理解
*/
public class demo7
{
    /*static int i=1;
    static
    {   //这个静态区域块只被执行一次
        i++;
    }
    public demo7()
    {
        i++;
    }*/
    public static void main(String []args)
    {
        /*demo7 d1=new demo7();
        System.out.println(d1.i);
        demo7 d2=new demo7();
        System.out.println(d2.i);*/
        Stu s1=new Stu(19,"mark1",9999);
        Stu s2=new Stu(20,"mark2",8978);
        System.out.println(Stu.Gettotal());
        
    }
}
```


​            
```java
class Stu
{
    //定义一个学生的类
    int age;
    String name;
    int fee;//学费
    static int totalfee;//定义一个静态变量为总的学费
    
    public Stu(int age,String name,int fee)
    {
        this.age=age;
        this.name=name;
        totalfee+=fee;
    }
    //返回总学费
    //这里是一个类方法，静态方法，把gettotal的值变成对象公共可以访问的
    //java里的规则类变量原则上用类方法去访问
    //类方法中不能访问非类变量
    public static int Gettotal()
    {
        return totalfee;
    }
    
}
```


什么时候使用类变量

定义学生类，或者钱的总数；用类变量属于公共的属性

类方法是和类相关的类的公共方法
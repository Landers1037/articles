---
title: java学习笔记4-this类
name: java-study4
date: 2018-07-26
id: 0
tags: [java,学习笔记]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


this.age=age;

this在java中表示当前的对象，它只能在类的方法中使用而不能在类定义的外部使用

```java
/*
休伯利安号
*/
public class demo5
{
    public static void main(String []args)
    {   Weapon weapon1=new Weapon("索尔之锤");
        Weapon weapon2=new Weapon("3rd圣遗物");
        Weapon weapon3=new Weapon("血犹大");
        Weapon weapon4=new Weapon("永暮双狼");
        Godess p1=new Godess(71,"卡莲 第六夜想曲");
        Godess p2=new Godess(71,"八重樱 逆神巫女");
        Godess p3=new Godess(70,"誓约 德丽莎.阿波卡利斯");
        Godess p4=new Godess(71,"白夜执事 符华");
        System.out.println("舰长：巧夺天工王大锤");
        System.out.println("UID:16884025");
        System.out.println("");
        p1.showInfo();
        weapon1.showInfo();
        p2.showInfo();
        weapon2.showInfo();
        p3.showInfo();
        weapon3.showInfo();
        p4.showInfo();
        weapon4.showInfo();
```


​            
```java
    }
}

class Godess
{   //每一个this都是直接对它所对应的对象的
    int level;
    String name;
    Weapon weapon;//引用类型
    
    public Godess(int level,String name)
    { //可读性不好
      this.level=level;
      this.name=name;
      this.weapon=weapon;
    }
    public void showInfo()
    {
        System.out.println("女武神名字："+this.name);
        System.out.println("女武神等级:"+this.level);
    }
}

class Weapon
{
    String name;
    public Weapon(String name)
    {
        this.name=name;
    }
    public void showInfo()
    {
        System.out.println("武器："+this.name);

    }
}
```

![](https://www.liaorenjie.top/wp-content/uploads/2018/07/god.jpg) 、例子1：

```java
/*
有一堆小睿智在玩堆雪人，不时有新的加入，请问如何知道此时有几个小睿智
面向对象的封装思想
*/
public class demo6
{
    public static void main(String []args)
    {
        int total=0;
        Child ch1=new Child(3,"pi1");
        ch1.join();
        Child ch2=new Child(5,"pi2");
        ch2.join();
        //total++;
        System.out.println("睿智total are:"+ch1.total);
        //这里读取ch1或者ch2的total值应该是一样的
    }
}
class Child
{
    int age;
    String name;
    static int  total=0;//静态变量
    public Child(int age,String name)
    {
        this.age=age;
        this.name=name;
    }
    public void join()
    {   total++;
        System.out.println("有一个小睿智加入了群聊");
    }
}
```

这里引入了静态变量static，直接在成员方法中定义，可以作为公共量被访问
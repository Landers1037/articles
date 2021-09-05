---
title: java学习笔记7 继承
name: java-study7
date: 2018-07-26
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


并不是所有的属性都可以继承

成员方法也可以继承

    /*继承练习2.18.7.25*/
    public class demo9{    
    public static void main(String []args)    
    {    
    }
    }
    //小学生class little{    
    private int age;    
    private String name;    
    private float fee;    
    //交费    
    public void pay(float fee)    {        
    this fee=fee;    }
    }
    // 大学生
    class big{    
    private int age;    
    private String name;    
    private float fee;    
    //交费    
    public void pay(float fee)    {        
    this fee=fee*0.8f;    
    }}//很明显有代码复用的问题
    
    class Stu
    {   //需要继承的不能定义为private
        public int age;
        public String name;
        public float fee;
        
        public void go()//可以被继承的方法
        {
        }
        
    }


​    
```java
/*
继承练习2
2.18.7.25
*/
public class demo9
{
    public static void main(String []args)
    {
    }
}
//将学生的共有的属性提取出来作为父类
class Stu
{   //需要继承的不能定义为private
    public int age;
    public String name;
    public float fee;
    
    public void go()
    {
    }
    
}
```


​        
```java
//小学生
class little extends Stu
{
    //交费
    public void pay(float fee)
    {
        this fee=fee;
    }
}
// 大学生
class big extends Stu
{
    public void pay(float fee)
    {
        this fee=fee*0.8f;
    }
}
//很明显有代码复用的问题
```

四种属性修饰符

public protected private 默认

除了private都可以被继承

如果不希望子类继承某个属性或者方法，就声明为私有的

特点：

子类最多继承一个父类

java的所有的类都是object的子类

方法重载

对于同一种方法，当其中的变量属性不一样时就会重载

注意：只是改变方法的修饰符，或者改变方法的返回值的数据类型（return float）都不会引起重载

方法覆盖

子类有一个方法和父类的方法，返回类型，参数一样，否则编译出错

注意修饰符最好一致，子类的方法不能缩小父类方法的访问权限，可以放大

方法的覆盖就是，继承父类里的方法时，在子类里面重新定义该方法，从而完成覆盖

```java
class father{    
public void call()    {        
System.out.print("father");    
{
}
class son extends father
{    
public void call()    {        
System.out.print("son");    
}
}
```
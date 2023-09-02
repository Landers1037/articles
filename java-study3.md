---
title: java学习笔记3-类和对象
name: java-study3
date: 2018-07-21
id: 0
tags: [java,学习笔记]
categories: []
abstract: ""
---


学习java的类和对象

类的名称最好用首字母大写的命名方式，而成员方法则最好使用小写的命名方式

在一个public的公共入口里面只能有一个类class

否则会出错

第一个test，建一个类为test

```java
public class test
{
    int age;
    String name;
    
    public static void main(String []args)
    {
        test t=new test();
        t.age=10;
        t.name="pipi";
        System.out.println("age is"+t.age);
        System.out.println("name is"+t.name);
    }
}
```

输出定义好的名字和年龄

第二个练习出现了许多的问题

```java
//java关于类和对象的练习
//2018.7.20
public class demo3
{
    
    public static void main(String []args)
    {

    Person p1=new Person();
        p1.age=10;
        p1.name="王pig";
        p1.speak();
        p1.jisuan();
        p1.jisuan(100);
        p1.add(12,10);
        System.out.println("age "+p1.age);
        System.out.println("name "+p1.name);

        //运行到这里跳出主函数，运行完毕

    }
}
//定义一个类为人类，确定他的属性
     class Person//类名首写字母大写
      {    int age;
        int hight;
        String name;
        //定义成员方法
        public void speak()//定义方法可以说话
        {
        System.out.print(" 我是你的父亲");
        }
        public void jisuan()//定义方法可以进行运算
        {
            int result=0;
            int i;
            for(i=0;i<1000;i++)
            {    
                result+=i;
            }
            System.out.println("计算结果"+result);
        }
        
        //带参数的成员方法
        public void jisuan(int n)
        {
            int result=0;
            for(int i=0;i<n;i++)
            {
                result+=i;
            }
            System.out.println("result="+result);
        }
        public void add(int num1,int num2)
        {
            int result=0;
            result=num1+num2;
            System.out.println("add is "+result);
            
        }
     }
```

出现问题 ![](https://www.liaorenjie.top/wp-content/uploads/2018/07/QQ截图20180720224712.jpg)![](https://www.liaorenjie.top/wp-content/uploads/2018/07/QQ图片20180720224839.png) 第一个在定义的add成员方法后面不小心加了=出错 第二个在public下只能有一个类，第一次在public class demo3的后面又加入了class person所以出了错误

> 建议成员变量用首字母大写的命名方式 成员方法用首字母小写
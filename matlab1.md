---
title: matlab矩阵基础1
name: matlab1
date: 2019-05-07
id: 0
tags: [matlab]
categories: []
abstract: ""
---


## 矩阵的逆

```matlab
>> A=[1,2,3;2,3,4;4,5,3];
>> B=inv(A)

B =

   -3.6667    3.0000   -0.3333
    3.3333   -3.0000    0.6667
   -0.6667    1.0000   -0.3333
```

## <!--more-->抽象求逆

```matlab
>> syms a b c d
>> A=[a b; c d];
>> B=A^(-1)
 
B =
 
[  d/(a*d - b*c), -b/(a*d - b*c)]
[ -c/(a*d - b*c),  a/(a*d - b*c)]
```

## 矩阵转置

```matlab
>> A=[2 0 -1;1 3 2]

A =

     2     0    -1
     1     3     2

>> B = [1 7 -1;4 2 3;2 0 1];
>> (A*B)'
% '表示转置符号
ans =

     0    17
    14    13
    -3    10
```

## 行列式计算

```matlab
>> C=[1,2;3,4];
>> det(C)

ans =

    -2
```

## 向量点积

```matlab
>> X=[ 1 2 3];
>> Y=[-3 -4 -5];
>> dot(X,Y)

ans =

   -26
   
%两个向量的维相同时，可以使用sum函数
>> sum(X.*Y)

ans =

   -26
```

## 混合积

```matlab
>> X=[1 2 3];
>> Y=[3 4 5];
>> Z=[5 6 7];
>> dot(X,cross(Y,Z))

ans =

     0
%根据混合积的性质，(axb)c=a(bxc)     
>> dot(cross(X,Y),Z)

ans =

     0
```


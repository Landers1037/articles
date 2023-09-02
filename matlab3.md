---
title: matlab的信号处理1
name: matlab3
date: 2019-05-14
id: 0
tags: [matlab]
categories: []
abstract: ""
---


## matlab2016a

### 建立全零矩阵基础

```matlab
>> a=zeros(2,5)

a =

     0     0     0     0     0
     0     0     0     0     0

>> a(:)=-4:5

a =

    -4    -2     0     2     4
    -3    -1     1     3     5

```

### <!--more-->建立变量

```matlab
%绘制y=1/(1+x)
>> syms x,y;
>> y=1/(x+1);
>> ezplot(y)
%可以传入参数，表示x的取值范围
ezplot(y,[1,100])
```

### 创建一个输入信号

```matlab
>> x=sin(2*pi*t)+sin(4*pi*t);
>>t=[1:199]./100;
>> plot(t,x)
```

### 绘制包络线

```matlab
clear;
t=0:0.01:2*pi;
y=sin(t).*sin(9*t);
[up,down] = envelope(y);  %包络线自定义函数
plot(t,y,'b');hold on;
plot(t,up,'r-.');
plot(t,down,'r-.');
xlabel('t'),ylabel('y')
hold off
```

### 学会for循环

```matlab
%找到100，200间的整除21的数字
for n=[100:200]
    m=rem(n,21);
    if(m==0)
        fprintf('the value of n is %d\n',n);
    end
end
```


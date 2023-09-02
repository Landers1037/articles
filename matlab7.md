---
title: matlab使用超松弛迭代法求解电容
name: matlab7
date: 2019-05-22
id: 0
tags: [matlab]
categories: []
abstract: ""
---


<font size=5>计算电磁学利用matlab超松弛迭代法求解微带线的电容</font>

### 题目

一块为正方形微带线 外围尺寸2cm，内部尺寸1cm，要求的电容误差10e-9，松弛系数1.9

设参数a,b,c,d分别为微带线的尺寸，n是迭代次数，tol为所需要的误差，rel为松弛迭代系数<!--more-->

```matlab
%第二次作业
function cap = capacitor(a,b,c,d,n,tol,rel)
h = 0.5*c/n;
na = round(0.5*a/h);
x= linspace(0,0.5*c,n+1);
m = round(0.5*d/h);
mb = round(0.5*b/h);
y = linspace(0,0.5*d,m+1);
%给定tol是电容的误差，rel是超松弛的参数

f = zeros(n+1,m+1); %全零矩阵
mask = ones(n+1,m+1)*rel; %全一矩阵

for i = 1:na+1 %对矩阵简化后求解
    for j = 1:mb+1
        mask(i,j) = 0;
        f(i,j) = 1;
    end
end
%高斯迭代法
oldcap = 0;
for iter = 1:1000
    f = seidel(f,mask,n,m); %计算迭代法得到松弛迭代后的矩阵
    cap = gauss(n,m,h,f); %计算电容
    if (abs(cap-oldcap)/cap<tol)
        %计算小于误差的时候得到电容的大小
        break
    else
        oldcap = cap;
    end
end
str = sprintf('number of iterations = %4i', iter);
disp(str) %输出最后的迭代次数
disp(cap)
end

function f = seidel(f,mask,n,m)
    for i = 2:n
        for j = 2:m
            f(i,j) = f(i,j) + mask(i,j)*(0.25*(f(i-1,j) + f(i+1,j) + f(i,j-1) + f(i,j+1)) - f(i,j));
        end
    end

    i = 1;
    for j = 2:m
        f(i,j) = f(i,j) + mask(i,j)*(0.25*(f(i+1,j) + f(i+1,j)+ f(i,j-1) + f(i,j+1)) - f(i,j));
    end

    j = 1;
    for i = 2:n
        f(i,j) = f(i,j) + mask(i,j)*(0.25*(f(i-1,j) + f(i+1,j)+ f(i,j+1) + f(i,j+1)) - f(i,j));
    end

end

function cap = gauss(n,m,h,f)
        q = 0;
        for i =1:n
            q = q + (f(i,m)+f(i+1,m))*0.5;
        end
        for j = 1:m
            q = q + (f(n,j)+f(n,j+1))*0.5;
        end

        cap = q*4;
        cap = cap*8.854187;
end
```

假设迭代次数为20次，在命令行运行

`capacitor(1,1,2,2,20,10e-9,1.9)`


---
title: matlab基础2
name: matlab2
date: 2019-05-08
id: 0
tags: [matlab]
categories: []
abstract: ""
---


## 计算1到100间的质数<!--more-->

```matlab
for i = 1000 : 1020  %外层循环，i的初值为2，终值为100
    for j = 1:1020  %内层循环，j的初值为2，终值为100
        if(~mod(i,j))  % i除以j取余后再取反
            break; % 跳出循环
        end
    end
    if(j > (i/j)) %检查是否有其他除数
        fprintf('%d is prime \n',i); %输出素数
    end
end

```


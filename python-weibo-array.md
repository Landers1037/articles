---
title: 使用numpy求解微波无源多端口网络的不定导纳矩阵
name: python-weibo-array
date: 2019-04-07
id: 0
tags: [numpy]
categories: []
abstract: ""
---


​	对于任意多任意端口网络的拓扑结构，如果通过输入来确定是不可能的因为任意多端口决定了不可能穷尽

​	考虑使用不定导纳矩阵的方法解决<!--more-->

## 不定导纳矩阵

把网络看做全部是由节点构成的，每个元件都连接两个节点，每个元件都有一个导纳矩阵。

假如对任意j节点，与其连接的有k个节点，那么就会存在k个导纳矩阵，对k个矩阵求和，就可以求出j节点的导纳矩阵

取节点1为公共节点，在求得的nxn矩阵中去掉第一行，第一列，最后得到的(n-1x(n-1)矩阵就是不定导纳矩阵，对不定导纳矩阵求逆得到Z矩阵

## 编程实现

算法如下

```python
import numpy as np
import time

#使用字典来存储数据
data = []
num = int(input('输入节点数'))
jiedian = int((num*(num-1)/2))
for n in range(1,jiedian+1):
    dict = {'导纳':'','l':'','r':''}
    G = int(input('导纳值电导'))
    B = int(input('导纳值电纳'))
    image = 1j
    dict['导纳'] = G + B*image
    dict['l'] = int(input('左节点'))
    dict['r'] = int(input('右节点'))
    data.append(dict)


def zero():
    #创建全0矩阵
    zarray = np.zeros((num,num),dtype=complex)
    return zarray

def buding(l,r,Y):
    zarray = zero()
    #单个元件的不定导纳矩阵
    zarray[l-1,l-1] = Y
    zarray[r-1,r-1] = Y
    zarray[l-1,r-1] = -Y
    zarray[r-1,l-1] = -Y
    # print(zarray,'\n')
    return zarray

def sum():
    #矩阵的相加

    sum = zero()
    for dict in data:
        array = buding(dict['l'],dict['r'],dict['导纳'])
        sum += array
    # print('求和矩阵')
    # print(sum)
    return sum

def result():
    #得到不定导纳矩阵
    budingjuzhen = sum()
    tmp1 = np.delete(budingjuzhen,0,axis=1)
    tmp2 = np.delete(tmp1,0,axis=0)
    budingjuzhen = tmp2
    # print('不定导纳矩阵\n')
    return budingjuzhen

def Zarray():
    #Z矩阵
    Z = np.linalg.pinv(result())  # 逆矩阵就是Z矩阵
    # print('逆矩阵\n', Z)
    return Z

time.sleep(2)
print('\n----开始----')

print('单个元件的导纳矩阵和\n',sum())
print('不定导纳矩阵\n',result())
print('逆矩阵\n', Zarray())

time.sleep(2)  
print('\n----结束----')
```

使用pyqt进行图形化重建
[请转到github](https://github.com/Landers1037/microwave_fuzhu)


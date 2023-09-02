---
title: cpu运算对应mips汇编指令集生成
name: cpu-mips-gen
date: 2018-04-14
id: 0
tags: [mips汇编学习,verilog]
categories: []
abstract: ""
---


### 利用mips模拟器qtspim将集合10种运算的汇编代码转换为机器码指令后输出

将最后语句的jump指令修改为对齐格式 最后放入文本文档修改后缀为coe文件 MEMORY\_INITIALIZATION\_RADIX=16; MEMORY\_INITIALIZATION\_VECTOR= 00432020, 8c440004, ac420008, 00831022, 00831025, 00831024, 0083102a, 10830002, 08100000, 8c620000, 08100000, 

在ISE平台新建文件 新建源文件为ip核选择刚才生成的coe文件导入

<!--more-->
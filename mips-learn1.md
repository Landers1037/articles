---
title: mips汇编-代码练习1
name: mips-learn1
date: 2018-03-29
id: 0
tags: [mips汇编学习]
categories: []
abstract: ""
---
**用汇编程序实现以下伪代码:要求采用移位指令实现乘除法运算**.  
<!--more-->


**用汇编程序实现以下伪代码:要求采用移位指令实现乘除法运算**.  <!--more-->

```c
Int main()

{

         Int K,Y;
         Int Z[50];
         Y = 56;
         For(k=0;k<50;k++)
            Z[K]=Y-16*(k/4+210);

}
```


** 要求将数组的50个值依次输出到屏幕上** **mips汇编代码**

```c
.data
z:.space 200  #分配50*4的空间存储数组
str1:.asciiz"z["
str2:.asciiz"]="
str3:.asciiz"\n"
.text
main:
  li $t0,0 #k
  li $s1,56 #y
  la $t1,z #address  #读取数组z的首地址

loop:
    slti $t2,$t0,50  #比较k<50 exit
    beq  $t2,$0,exit #判断条件满足则执行exit
    srl $t3,$t0,2    #运算k/2
    addi $s2,$t3,210 #运算k/4+210
    sll $s2,$s2,4    #运算16*（k/4+210）
    sub $s2,$s1,$s2  #运算y-16*（k/4+210）
    sw $s2,0($t1)    #将数组的运算结果写入内存
    
    li $v0,4
    la $a0,str1	     #z[
    syscall
    li $v0,1
    add $a0,$t0,$0
    syscall          #k的值
    li $v0,4
    la $a0,str2
    syscall          #]=
    li $v0,1
    lw $a0,0($t1)
    syscall          #输出的z值
    li $v0,4
    la $a0,str3      #\n回车换行
    syscall
    addi $t0,$t0,1   #k-1
    addi $t1,$t1,4   #address+4
    j loop
exit:
    li $v0,10
    syscall
```


仿真结果 ![](/images/mips-learn-1-1.webp)![9vAFmQ.png](/images/mips-learn-1-2.webp) **错误总结**

```c
.data
z:.space 200
str1:.asciiz"z["
str2:.asciiz"]"
str3:.asciiz"\n"
.text
main:
  li $t0,0 #k
  li $s1,56 #y
  la $t1,z #address

loop:
   
    srl $t3,$t0,2  #k/2
    addi $s2,$t3,210   #k/4+210
    sll $s2,$s2,4    #16*
    sub $s2,$s1,$s2 	#y-16*
    sw $s2,0($t1) 
    li $v0,4
    la $a0,str1	#z[
    syscall
    li $v0,1
    add $a0,$t0,$0
    syscall #k
    li $v0,4
    la $a0,str2
    syscall #]
    li $v0,1
    lw $a0,0($t1)
    syscall #output numbers
    li $v0,4
    la $a0,str3#z[
    syscall
    addi $t0,$t0,1 #k-1
    addi $t1,$t1,4  #address+4
```
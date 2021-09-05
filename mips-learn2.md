---
title: mips汇编语言练习
name: mips-learn2
date: 2018-03-27
id: 0
tags: [mips汇编学习,汇编]
categories: []
abstract: ""
---


1.显示16进制数字

<!--more-->


1.显示16进制数字
<!--more-->


```verilog
.data#显示16进制数字
str:byte 0x20,0x30,0x78
hex:.space 8
.byte 0x00
.text
main:
     li $v0,0x12AF5678
	 la $s0,hex
	 addi $s0,$s0,7
	 li $s1,8
rep:andi $s2,$v0,0xf
    addi $s2,$s2,0x30
	slti $t0,$s2,0x3a
	bne $t0,$0,less
	addi $s2,$s2,0x07
less:sb $s2,0($s0)
     addi $s0,$s0,-1
	 addi $s1,$s1,-1
	 srl $v0,$v0,4
	 slti $t0,$s1,1
	 beq $t0,$0,rep
	 li $v0,4
	 la $a0,str
	 syscall
	 li $v0,10
	 syscall
2.输入16进制数字	
```


```verilog
.data#input 16进制数字
str:.space 11
err:asciiz "input error"
.text
main:li $v0,8
	la $a0,str
	li $s1,11
	syscall
	lbu $t1,0($a0)
	li $t0,0x30
	bne $t1,$t0,printerr
	lbu $t1,1($a0)
	li $t0,0x78
	bne $t1,$t0,printer
	addi $a0,$a0,2
	li $v0,0
rep:lb $t0,0($a0)
	li $t2,0x0a
	beq $t0,$t2,exit
	addi $t0,$t0,-48
	slti $t1,$t0,0
	bne $t1,$0,printerr
	slti $t1,$t0,10
	beq $t1,$0,upper
	j convert
upper:addi $t0,$t0,-7
	  slti $t1，$t0,10
	  bne $t1,$0,printerr
	  slti $t1,$t0,16
	  beq $t1,$0,lower
	  j convert
lower:addi $t0,$t0,-32
	  slti $t1,$t0,10
	  bne $t1,$0,printerr
	  slti $t1,$t0,16
	  beq $t1,$0,printerr
convert:sll $v0,$v0,4
	   or $v0,$v0,$t0
	   addi $a0,$a0,1
	   j rep
printerr:li $v0,4
		 la $a0,err
		 syscall
exit :li $v0,10
		syscall
```


3.显示10进制无符号数字

```verilog
.data#显示十进制数无符号数
str:.space 10
null:     .byte 0
.text 
main: li $a0,0xffffffff
	la $t0,10
	la $s0,null
rep:addi $s0,$s0,-1
	divu $a0,$t0
	mfhi $t1
	addi $t1,$t1,0x30
	sb $t1,0($s0)
	mflo $a0
	beq $a0,$0,print
	j rep
print: add $a0,$s0,$0
     li $v0,4
	 syscall
	 li $v0,10
     syscall
```


4.显示十进制符号数字

```verilog
.data#signed 10
str:.space 11
null: .byte 0
.text
main: li $a0,0xffffffff
      add $a1,$a0,$0
	  slt $t1,$a0,$0
	  beq $t1,$0,posi
	  addi $a0,$a0,-1
	  li $t2,0xffffffff
	  xor $a1,$a0,$t2
posi:la $t0,10
     la $s0,null
rep:addi $s0,$s0,-1
    divu $a1,$t0
	mfhi $t1
	addi $t1,$t1,0x30
	sb $t1,0($s0)
	mflo $a1
	beq $a1,$0,print
	j rep
      	  
print:slt $t1,$a0,$0
      beq $t1,$0,printposi
	  addi $s0,$s0,-1
	  li $t1,0x2d
	  sb $t1,0($s0)
printposi: add $a0,$s0,$0
          li $v0,4
		  syscall
		  li $v0,10
		  syscall
```

5.输入10进制数

```verilog
.data#input 10 count
str:.space 12
err:.asciiz"input error"
.text
main: li $v0,8
      la $a0,str
	  li $a1,12
	  syscall
	  lbu $t1,0($a0)
	  li $t0,0x2d
	  bne $t1,$t0,posi
	  addi $a0,$a0,1
posi:li $v1,0
     li $t2,10
rep:lb $t0,0($a0)
    li $t2,0x0a
	beq $t0,$t2,exit
	addi $t0,$t0,-48
	slti $t1,$t0,0
	bne $t1,0,printerr
	slti $t1,$t0,10
	beq $t1,$0,printerr
convert:multu $v1,$t2
        mflo $v1
		mfhi $t2
		bne $t2,0,printerr
		add $v1,$v1,$t0
		addi $a0,$a0,1
		j rep
printerr:li $v0,4
         la $a0,err
		 syscall
exit:la $a0,str
     lbu $t1,0($a0)
	 li $t0,0x2d
	 bne $t1,$t0,return
	 li $t2,0xffffffff
	 xor $v1,$v1,$t2
	 addi $v1,$v1,1
return: li $v0,10
        syscall
```

6.对数组的正数和负数求和

```verilog
.data#data count
array:.word -1,3,4,-5
posi:.asciiz"\nthe sun of positive numbers are:"	 
nega:.asciiz"\nthe sun of negative numbers are:"
.text
main:
     li $v0,4
	 la $a0,posi
	 syscall
	 la $a0,array
	 li $a1,4
	 jal sum
	 add $a0,$v0,$0
	 li $v0,1
	 syscall
	 li $v0,4
	 la $a0,nega
	 syscall
	 add $a0,$v1,$0
	 li $v0,1
	 syscall
	 li $v0,10
	 syscall
	 
sum:li $v0,0
    li $v1,0
loop:
    blez $a1,retzz
    addi $a1,$a1 ,-1
    lw $t0,0($a0)
    addi $a0,$a0,4
    bltz $t0,negg
    add $v0,$v0,$t0
    b loop
negg:add $v1,$v1,$t0
     b loop
retzz: jr $ra
```
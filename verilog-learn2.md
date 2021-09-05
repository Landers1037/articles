---
title: verilog语法之reg和wire区别
name: verilog-learn2
date: 2018-03-10
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---


​	reg相当于存储单元，wire相当于物理连线 Verilog 中变量的物理数据分为线型和寄存器型。

​	这两种类型的变量在定义时要设置位宽，缺省为1位。变量的每一位可以是0，1，X，Z。其中x代表一个未被预置初始状态的变量或者是由于由两个或多个驱动装置试图将之设定为不同的值而引起的冲突型线型变量。z代表高阻状态或浮空量。 
<!--more-->


​	reg相当于存储单元，wire相当于物理连线 Verilog 中变量的物理数据分为线型和寄存器型。

​	这两种类型的变量在定义时要设置位宽，缺省为1位。变量的每一位可以是0，1，X，Z。其中x代表一个未被预置初始状态的变量或者是由于由两个或多个驱动装置试图将之设定为不同的值而引起的冲突型线型变量。z代表高阻状态或浮空量。 <!--more-->
​	线型数据包括wire,wand,wor等几种类型在被一个以上激励源驱动时，不同的线型数据有各自决定其最终值的分辨办法。 
​	两者的区别是：即存器型数据保持最后一次的赋值，而线型数据需要持续的驱动 输入端口可以由net/reg驱动，但输入端口只能是net；输出端口可以使net/reg类型，输出端口只能驱动net；若输出端口在过程块中赋值则为reg型，若在过程块外赋值则为net型 用关键词inout声明一个双向端口, inout端口不能声明为寄存器类型，只能是net类型。 wire表示直通，即只要输入有变化，输出马上无条件地反映；reg表示一定要有触发，输出才会反映输入。 不指定就默认为1位wire类型。专门指定出wire类型，可能是多位或为使程序易读。wire只能被assign连续赋值，reg只能在initial和always中赋值。wire使用在连续赋值语句中，而reg使用在过程赋值语句中。 在连续赋值语句中，表达式右侧的计算结果可以立即更新表达式的左侧。
​	在理解上，相当于一个逻辑之后直接连了一条线，这个逻辑对应于表达式的右侧，而这条线就对应于wire。在过程赋值语句中，表达式右侧的计算结果在某种条件的触发下放到一个变量当中，而这个变量可以声明成reg类型的。根据触发条件的不同，过程赋值语句可以建模不同的硬件结构：如果这个条件是时钟的上升沿或下降沿，那么这个硬件模型就是一个触发器；如果这个条件是某一信号的高电平或低电平，那么这个硬件模型就是一个锁存器；如果这个条件是赋值语句右侧任意操作数的变化，那么这个硬件模型就是一个组合逻辑。 输入端口可以由wire/reg驱动，但输入端口只能是wire；输出端口可以使wire/reg类型，输出端口只能驱动wire；若输出端口在过程块中赋值则为reg型，若在过程块外赋值则为net型。用关键词inout声明一个双向端口, inout端口不能声明为reg类型，只能是wire类型；输入和双向端口不能声明为寄存器类型。 

简单来说硬件描述语言有两种用途：1、仿真，2、综合。 
对于wire和reg，也要从这两个角度来考虑。
从仿真的角度来说，HDL语言面对的是编译器（如Modelsim等），相当于软件思路。 这时： wire对应于连续赋值，如assign reg对应于过程赋值，如always，initial 

从综合的角度来说，HDL语言面对的是综合器（如DC等），要从电路的角度来考虑。 这时： 
1、wire型的变量综合出来一般是一根导线；
2、reg变量在always块中有两种情况：
	(1)、always后的敏感表中是（a or b or c）形式的，也就是不带时钟边沿的，综合出来还是组合逻辑 
	(2)、always后的敏感表中是（posedge clk）形式的，也就是带边沿的，综合出来一般是时序逻辑，会包含触发器（Flip－Flop） 在设计中，输入信号一般来说你是不知道上一级是寄存器输出还是组合逻辑输出，那么对于本级来说就是一根导线，也就是wire型。而输出信号则由你自己来决定是寄存器输出还是组合逻辑输出，wire型、reg型都可以。但一般的，整个设计的外部输出（即最顶层模块的输出），要求是寄存器输出，较稳定、扇出能力也较好。 
转自csdn http://blog.csdn.net/zmq5411/article/details/7952461
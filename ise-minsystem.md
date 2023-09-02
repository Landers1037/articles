---
title: 使用ISE平台建立最小系统
name: ise-minsystem
date: 2018-05-17
id: 0
tags: [学习笔记,技术]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


### 使用Xilinx ISE平台

在平台的tools文件夹下找到**xilinx platform studio**打开
![](/images/ise-minsystem-1.webp) 

在本地建立一个文件夹将生成的**xmp**文件放在指定的文件夹内 本次开发使用**nexy4**板，选择参数为**artix7，speed grade**为**-1**其余的数值均不改变 类别选择单处理器系统然后点击**next** 如下界面
![](/images/ise-minsystem-2.webp) 
设置参数，选择**32kb**的内存空间，然后点击add device从i\\o串口类型下选择**UART**类型，设备名称为**RS232**，然后点击**ok**
![](/images/ise-minsystem-3.webp) 
此时进入主界面，在这里的ports块下可以看见我们建立的接口
![](/images/ise-minsystem-4.webp) )
因为我们只需要单一的时钟源而nexy4提供的是两个方向相反的时钟源所以需要对时钟进行修改。 
![](/images/ise-minsystem-5.webp) 

1.  在port选项卡中的External port下找到CLK\_N,CLK\_P右键delete
2.  在下面找到clock\_generator\_0中的CLKIN，右键选中选择make external
3.  然后我们回到之前的External port里面，找到RS232的I接口，前面的名字修改为RsRx
4.  RS232的O接口，前面的名字修改为RsTx
5.  注意i和o的顺序不要搞反，不然后面的管脚约束会出问题

修改后为如下
![](/images/ise-minsystem-6.webp) 
现在我们建立引脚的约束 在左下角找到project进入 
![](/images/ise-minsystem-7.webp)
找到ucf文件点击修改
![](/images/ise-minsystem-8.webp) 
添加约束代码


    NET "CLK" TNM_NET = sys_clk_pin;
    TIMESPEC TS_sys_clk_pin = PERIOD sys_clk_pin 100000 kHz;
    NET"CLK"  LOC="E3"|IOSTANDARD ="LVCMOS33";
    NET"RESET"  LOC="E16"|IOSTANDARD ="LVCMOS33";
    NET"RsRx"  LOC="C4"|IOSTANDARD ="LVCMOS33";
    NET"RsTx"  LOC="D4"|IOSTANDARD ="LVCMOS33";

保存

#### **接下来使用SDK设计程序输出最小系统到程序**

在xps的主界面选择project>export hardware design to SDK，选择export&launch按钮
![](/images/ise-minsystem-9.webp) 

在工程文件夹中建立一个用来放SDK的文件夹，选择该路径然后点击ok 在主界面file下选择new，xilinx board support package并设置参数 选择standalone后点击finish 接下来弹出窗口，选择ok 进入创建窗口，依次点击file下的new，application project选项 在Board support package项选择Using existing language选择c语言然后点击next，选择hello world作为模板，点击finish

#### 到这里实现了建立最小系统，接下来需要在SDK里写入c语言程序然后在我们的系统里运行。
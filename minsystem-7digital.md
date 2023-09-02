---
title: 基于最小系统—八位7段数码管平移程序
name: minsystem-7digital
date: 2018-05-18
id: 0
tags: [学习笔记,技术]
categories: []
abstract: ""
---


基于我们建立的最小系统，设计7段数码管的显示程序

1.  创建工程，建立一个工程的文件夹命名为seg_7
2.  启动XPS，打开system.xmp文件
3.  添加GPIO的ip内核

GPIO内核的创建步骤 在界面的下方找到system assemly view窗口

![](http://file.mgek.cc/images/blog/minsystem-7digital-1.webp) 
在左下角的ip catalog中找到GPIO模块，添加一个外设如图

![](http://file.mgek.cc/images/blog/minsystem-7digital-2.webp) 
![](http://file.mgek.cc/images/blog/minsystem-7digital-3.webp)  
然后双击AXI General Purpose I/O，点击yes 选择上面的选项，将时钟与端口连接 在port里找到我们刚才建立的gpio接口，将名字命名为seg_7 
![](http://file.mgek.cc/images/blog/minsystem-7digital-4.webp) 
接下来双击seg_7进行配置
默认不使用中断采取延时的方式进行动态扫描 如下为管脚约束（因为只是使用4位数码管，所以对应的gpio也是4位输出）


```verilog
NET "CLK" TNM_NET = sys_clk_pin;
TIMESPEC TS_sys_clk_pin = PERIOD sys_clk_pin 100000 kHz;
NET"CLK"  LOC="E3"|IOSTANDARD ="LVCMOS33";
NET"RESET"  LOC="E16"|IOSTANDARD ="LVCMOS33";
NET"RsRx"  LOC="C4"|IOSTANDARD ="LVCMOS33";
NET"RsTx"  LOC="D4"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<0>"  LOC="L3"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<1>"  LOC="N1"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<2>"  LOC="L5"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<3>"  LOC="L4"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<4>"  LOC="K3"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<5>"  LOC="M2"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<6>"  LOC="L6"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO2_IO_pin<7>"  LOC="M4"|IOSTANDARD ="LVCMOS33";

NET"seg_7_GPIO_IO_pin<0>"  LOC="N6"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<1>"  LOC="M6"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<2>"  LOC="M3"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<3>"  LOC="N5"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<4>" LOC="N2"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<5>" LOC="N4"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<6>" LOC="L1"|IOSTANDARD ="LVCMOS33";
NET"seg_7_GPIO_IO_pin<7>" LOC="M1"|IOSTANDARD ="LVCMOS33";
```

接下来生成bit流文件后点击输出到sdk即可进行c代码的编写
![](http://file.mgek.cc/images/blog/minsystem-7digital-5.webp) 
SDK的使用详细见教科书 提供c语言代码

```verilog
/*
 * gpio.c
 *
 *  Created on: 2018-5-20
 *      Author: Landers 皮
 */
#include"xparameters.h"
#include "xil_io.h"
int main()
{   char segcode[8]={0x92,0x92,0x92,0x92,0x92,0x92,0x92,0x92};
    int i,j;
    int pos=0x7f;//初始位码
    Xil_Out8(XPAR_SEG_7_BASEADDR+0x4,0x0);//gpio输出
    Xil_Out8(XPAR_SEG_7_BASEADDR+0xc,0x0);//gpio2 output
    while(1)

    {	for(i=0;i<8;i++)

    {
    Xil_Out8(XPAR_SEG_7_BASEADDR,pos);//输出位码
    Xil_Out8(XPAR_SEG_7_BASEADDR+0x8,segcode[i]);//输出段码
    for(j=0;j<0xa5e10;j++);//delay
    	pos=pos>>1;
	}
	pos=0x7f;
}

	/* code */
	return 0;
}
```


功能实现四位数码管动态显示数码，已经初始化定义输出为s，默认时间为100mhz太长，已经修改加快。 实验的验收要求使用调用api函数使用中断的方式实现数码管的位移 此时需要添加timer的ip内核和intc中断的ip内核 程序代码如下：

```verilog
#include "xparameters.h"
#include "xtmrctr.h"
#include "xintc.h"
#include "xil_exception.h"
#define TMRCTR_DEVICE_ID   XPAR_TMRCTR_0_DEVICE_ID
#define TMRCTR_INTERRUPT_ID  XPAR_INTC_0_TMRCTR_0_VEC_ID
#define INTC_DEVICE_ID    XPAR_INTC_0_DEVICE_ID
#define TIMER_CNTR_0  0
#define RESET_VALUE  0X5F5E100
void TimerCounterHandler(void *CallBackRef, u8 TmrCtrNumber);
XIntc  InterruptController;
XTmrCtr TimerCounterInst;
u8 seg_7;

int main(void) {
	int Status;
	seg_7 = 0x7f;
	int i=0;
	Xil_Out8(0x40000004,0x0);
	Xil_Out8(0x4000000c,0x0);
	Status = XTmrCtr_Initialize(&TimerCounterInst, TMRCTR_DEVICE_ID);
	XIntc_Initialize(&InterruptController, INTC_DEVICE_ID);
	XIntc_Connect(&InterruptController,INTC_DEVICE_ID,
			(XInterruptHandler)XTmrCtr_InterruptHandler,
			(void *)&TimerCounterInst);//注册中断服务程序
	XIntc_Start(&InterruptController, XIN_REAL_MODE);
	XIntc_Enable(&InterruptController, TMRCTR_INTERRUPT_ID);
	microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,
			&InterruptController);//
	microblaze_enable_interrupts();
	XTmrCtr_SetHandler(&TimerCounterInst, TimerCounterHandler, &TimerCounterInst);//注册定时器中断服务程序
	XTmrCtr_SetOptions(&TimerCounterInst, TIMER_CNTR_0,
			XTC_INT_MODE_OPTION | XTC_AUTO_RELOAD_OPTION | XTC_DOWN_COUNT_OPTION);
	XTmrCtr_SetResetValue(&TimerCounterInst, TIMER_CNTR_0, RESET_VALUE);
	XTmrCtr_Start(&TimerCounterInst, TIMER_CNTR_0);
	while(1);
	return XST_SUCCESS;

}

void TimerCounterHandler(void *CallBackRef, u8 TmrCtrNumber) {
	Xil_Out8(0x40000000, seg_7 );
	Xil_Out8(0x40000008,0xf9);
	seg_7=seg_7>>1;
	seg_7=seg_7+0x80;
	int i=0;
	i++;
    if(i==8|seg_7==0xff){
    seg_7=0x7f;
    i=0;
      }
}
```


**特别注意**

>         在数码管的向右位移时为整段位移所以上一位数码管的值不会清零，此时在循环语句中添加位码+0x80的语句实现首位的数值进位清零来实现。 例如初始位码为0x7f即01111111，此时第一个数码管亮其余的熄灭，当右移1位时变为00111111（右移时前一位补0参考微机原理），所以此时的灯显示为左边两个都亮。我们要使每次只有一位灯亮即每次位码里只有一个0其他全部是1。 所以我们在右移的语句后面添加位码+0x80的操作，即+10000000.此时每次右移后的0会被改为1，整体平移时会跟随移动。这样01111111的下一次右移就是10111111，满足只有一个灯亮。 在最后的判断语句里当位码为0xff即11111111时应该为所有的灯都熄灭时，此时做循环判定，回到初始位码重新循环。 这里的最终循环判断语句中，使用i=8或位码为0xff来实现。![](http://file.mgek.cc/images/blog/minsystem-7digital-6.webp)  硬件和软件的实现原理分析 1.程序中我们一共有三个handler程序，其中用到的定时器作为产生中断请求的中断源，所以定时器注册的中断服务程序是我们自己需要编写的程序。 2.当中断请求发送给中断控制器，中断控制器会响应中断请求，并且由系统生成一个中断控制器的中断服务程序我们称为控制器服务程序。这个程序里会自动添加产生中断的设备的地址，并且根据系统自己生成的程序查询外部此设备是否产生中断。 3.接着中断控制器会将中断请求信号传送给microblaze的cpu处理，当cpu接收到中断，会根据系统事先生成的中断服务程序（我们称为cpu中断服务）来查询中断的地址，即此程序可以查询中断控制器的中断服务程序，再由中断控制器的服务程序查找到中断源（即timer）的中断请求，并且最终实现定时器中断服务程序
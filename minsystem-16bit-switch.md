---
title: 基于最小系统—16位按键开关拨动输出
name: minsystem-16bit-switch
date: 2018-05-22
id: 0
tags: [学习笔记,技术]
categories: []
abstract: ""
---


```verilog
#include "xparameters.h"
#include "xgpio.h"
#include "xintc.h"
#include "stdio.h"
void Initialize();
void Delay_50ms();
void PushBtnHandler(void *CallBackRef);
void SwitchHandler(void *CallBackRef);
XGpio Btns,Dips;
XIntc intCtrl;
int pshBtn,pshDip;
int state1,state2;
int main()
{
	Initialize();
	xil_printf("\r\nRunning GpioInterrupt!\r\n");
	while(1)
	{
		if(pshBtn)
		{
			xil_printf("ButtonInterrupt! 0x%x\n\r",state1);
			pshBtn=0;
		}
		if(pshDip)
		{
				if(state2>=0x8000)
				xil_printf("SwitchInterrupt! 1111\n\r");
				else if(state2>=0x4000)
				xil_printf("SwitchInterrupt! 1110\n\r");
				else if(state2>=0x2000)
				xil_printf("SwitchInterrupt! 1101\n\r");
				else if(state2>=0x1000)
				xil_printf("SwitchInterrupt! 1100\n\r");
				else if(state2>=0x0800)
				xil_printf("SwitchInterrupt! 1011\n\r");
				else if(state2>=0x0400)
				xil_printf("SwitchInterrupt! 1010\n\r");
				else if(state2>=0x0200)
				xil_printf("SwitchInterrupt! 1001\n\r");
				else if(state2>=0x0100)
				xil_printf("SwitchInterrupt! 1000\n\r");
				else if(state2>=0x0080)
				xil_printf("SwitchInterrupt! 0111\n\r");
				else if(state2>=0x0040)
				xil_printf("SwitchInterrupt! 0110\n\r");
				else if(state2>=0x0020)
				xil_printf("SwitchInterrupt! 0101\n\r");
				else if(state2>=0x0010)
				xil_printf("SwitchInterrupt! 0100\n\r");
				else if(state2>=0x0008)
				xil_printf("SwitchInterrupt! 0011\n\r");
				else if(state2>=0x0004)
				xil_printf("SwitchInterrupt! 0010\n\r");
				else if(state2>=0x0002)
				xil_printf("SwitchInterrupt! 0001\n\r");
				else if(state2>=0x0001)
				xil_printf("SwitchInterrupt! 0000\n\r");
				pshDip=0;
		}
	}
	return 0;
}

//初始化
void Initialize()
{
	//初始化dips，并将其设置为输入
	XGpio_Initialize(&Dips,XPAR_DIP_DEVICE_ID);
	XGpio_SetDataDirection(&Dips,1,0xff);
	//初始化btns，并将其设置为输入
	XGpio_Initialize(&Btns,XPAR_BUTTON_DEVICE_ID);
	XGpio_SetDataDirection(&Btns,1,0xff);
	//初始化intctrl
	XIntc_Initialize(&intCtrl,XPAR_AXI_INTC_0_DEVICE_ID);
	//GPIO中断使能
	XGpio_InterruptEnable(&Btns,1);
	XGpio_InterruptGlobalEnable(&Btns);
	XGpio_InterruptEnable(&Dips,1);
	XGpio_InterruptGlobalEnable(&Dips);
	//中断控制器进行中断源使能
	XIntc_Enable(&intCtrl,XPAR_AXI_INTC_0_BUTTON_IP2INTC_IRPT_INTR);
	XIntc_Enable(&intCtrl,XPAR_AXI_INTC_0_DIP_IP2INTC_IRPT_INTR);
	//注册中断服务函数
	XIntc_Connect(&intCtrl,XPAR_AXI_INTC_0_BUTTON_IP2INTC_IRPT_INTR,(XInterruptHandler)PushBtnHandler,(void*)0);
	XIntc_Connect(&intCtrl,XPAR_AXI_INTC_0_DIP_IP2INTC_IRPT_INTR,(XInterruptHandler)SwitchHandler,(void*)0);
	//允许中断处理器处理中断
	microblaze_enable_interrupts();
	//注册中断控制器处理函数
	microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,(void*)&intCtrl);
	//启动中断控制器
	XIntc_Start(&intCtrl,XIN_REAL_MODE);
}

//延时控制
void Delay_50ms()
{
	int i;
	for(i=0;i<5000000;i++);
}
//按键中断服务程序
void PushBtnHandler(void *CallBackRef)
{
	state1=XGpio_DiscreteRead(&Btns,1);//读取按键状态
	pshBtn=1;
	XGpio_InterruptDisable(&Btns,1);//暂时禁止button中断
	Delay_50ms();//延时50ms去抖动
	XGpio_InterruptClear(&Btns,1);//清除中断标志位
	XGpio_InterruptEnable(&Btns,1);//再次开放按键中断
}
//开关中断服务程序
void SwitchHandler(void *CallBackRef)
{
	state2=XGpio_DiscreteRead(&Dips,1);//读取开关状态
	pshDip=1;
	XGpio_InterruptClear(&Dips,1);//清除中断标志位
}
```


感谢wxk大佬的无私提供


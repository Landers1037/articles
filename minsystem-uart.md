---
title: 基于最小系统—UART串口设计
name: minsystem-uart
date: 2018-06-18
id: 0
tags: [学习笔记,技术,最小系统]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


基于最小系统的UART实验，要求实现双板之间的串口信息通信 使用Xilinx的NEXY4板 提供两种程序代码 1.使用attribute的io函数

```verilog
#include "xparameters.h"
#include"xintc.h"
#include"xil_io.h"
#include"xil_types.h"
#include"mb_interface.h"
#include"stdio.h"
#define dip_data 0x00
#define dip_tri 0x04
#define dip_isr 0x0120

#define led_data 0x00
#define led_tri 0x04
#define led_isr 0x0120

#define intc_isr 0x00
#define intc_ier 0x08
#define intc_iar 0x0c
#define intc_mer 0x1c

#define uart_rx 0x00
#define uart_tx 0x04
#define uart_stat 0x08
#define uart_ctl 0x0c

#define gpio_base 0x40000000//gpio ,uart address
#define led 0x40040000
#define intc_base 0x41200000
#define uart_base 0x40640000

void My_ISR(void)__attribute__((interrupt_handler));
void Initialize();
void Uarthandler();
void delay_50ms();

short Rx_flag;
unsigned char rxData;
int txtime;

int main(void)
{
	xil_printf("\r\n running test\r\n");
	Initialize();
	while(1)
	{
		if(Rx_flag)
		{   xil_printf("the rx data is0x%x\r\n",rxData);
		    Rx_flag=0;

	}
	delay_50ms();
	txtime=txtime+1;
	if(txtime==10000) txtime=0x00;
		int uart_flag;
		uart_flag=Xil_In32(uart_base+uart_stat);
		if(uart_flag&0x04)
			{
				if(txtime%2)
					{   int txdata;
						txdata=Xil_In32(gpio_base+dip_data);
						Xil_Out32(uart_base+uart_tx,txdata);
						}
			}
}
return 0;
}
void Initialize()
 {
 	Rx_flag=0x00;
 	rxData=0x00;
 	txtime=0x00;
 	Xil_Out8(gpio_base+dip_tri,0xff);
 	Xil_Out8(led+led_tri,0x00);
 	Xil_Out8(led+led_data,0x5a);

 	Xil_Out32(uart_base+uart_ctl,0x13);
 	Xil_Out32(intc_base+intc_iar,0xffffffff);
 	Xil_Out32(intc_base+intc_ier,0x80000003);
    Xil_Out32(intc_base+intc_mer,0x03);
    microblaze_enable_interrupts();
```


​    
```verilog
 }
void My_ISR(void)
{
 int status;
 status=Xil_In32(intc_base+intc_isr);
 if(status&0x01)
 	{
 		Uarthandler();

 	}
 	Xil_Out32(intc_base+intc_iar,status);
}

void Uarthandler()
{   int uart_flag;
    uart_flag=Xil_In32(uart_base+uart_stat);
	if(uart_flag&0x01)
		{
			Rx_flag=1;
			rxData=Xil_In32(uart_base+uart_rx);
			Xil_Out8(led+led_data,rxData);

		}
		Xil_Out32(uart_base+uart_ctl,0x13);

}

void delay_50ms()
{
	int i;
	for(i=0;i<5000000;i++);

}
```


2.采用api函数

```verilog
#include "xparameters.h"
#include "xintc.h"
#include "xil_io.h"
#include "xil_types.h"
#include "mb_interface.h"
#include "xgpio.h"
#include "xuartlite.h"
#include "xuartlite_l.h"
```


​    
```verilog
void Initialize();
void SwitchHandler(void *CallBackRef);
void RecvHandler();
void SendHandler();
void Delay(int n);

XGpio Dips,Leds;
XUartLite uart;
XIntc intCtrl;

u8 rxBuffer[2];
u8 txBuffer[2];
u16 rxData;
u16 txData;
```


​    
```verilog
int main()
{
	xil_printf("\r\nRunning UART Test !\r\n");
	Initialize();
	while(1);
	return 0;
}

void Initialize()
{

	rxBuffer[0]=rxBuffer[1]=txBuffer[0]=txBuffer[1]=0;
	XGpio_Initialize(&Dips,XPAR_DIP_DEVICE_ID);
	XGpio_SetDataDirection(&Dips,1,0x0ff);
	XGpio_Initialize(&Leds,XPAR_LED_DEVICE_ID);
	XGpio_SetDataDirection(&Leds,1,0x0);
	XUartLite_Initialize (&uart,XPAR_AXI_UARTLITE_0_DEVICE_ID);
	XUartLite_SelfTest(&uart);
	XIntc_Initialize(&intCtrl,XPAR_AXI_INTC_0_DEVICE_ID);

	XUartLite_ResetFifos(&uart);

	XGpio_InterruptEnable(&Dips,1);
	XGpio_InterruptGlobalEnable(&Dips);

	XUartLite_EnableInterrupt(&uart);
	XIntc_Enable(&intCtrl,XPAR_AXI_INTC_0_DIP_IP2INTC_IRPT_INTR);
	XIntc_Enable(&intCtrl,XPAR_AXI_INTC_0_AXI_UARTLITE_0_INTERRUPT_INTR);
	XIntc_Start(&intCtrl,XIN_REAL_MODE);
	microblaze_enable_interrupts();
```


​    
    
```verilog
	XIntc_Connect(&intCtrl,XPAR_AXI_INTC_0_DIP_IP2INTC_IRPT_INTR,(XInterruptHandler)SwitchHandler,(void*)0);
	XIntc_Connect(&intCtrl,XPAR_AXI_INTC_0_AXI_UARTLITE_0_INTERRUPT_INTR,(XInterruptHandler)XUartLite_InterruptHandler,&uart);
	XUartLite_SetRecvHandler (&uart,RecvHandler,&uart);
	XUartLite_SetSendHandler(&uart, SendHandler, &uart);
	microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,(void*)&intCtrl);
}

void RecvHandler()
{
				Delay(5);
				XUartLite_Recv (&uart,rxBuffer,2);
				rxData=rxBuffer[0]+(rxBuffer[1]<<8);
				XGpio_DiscreteSet (&Leds,1,rxData);
				xil_printf("UART receives data ! 0x%x\r\n",rxData);

}

void SwitchHandler(void *CallBackRef)
	{

		txData=XGpio_DiscreteRead(&Dips,1);
		XGpio_InterruptClear(&Dips,1);
		txBuffer[0]=(u8)txData;
		txBuffer[1]=(u8)(txData>>8);

		Delay(5);
		XUartLite_Send(&uart,txBuffer,2);
	}
void SendHandler()
{
	xil_printf("UART sends data ! 0x%x\r\n",txData);
}
void Delay(int n)
{
	int i;
	for(i=0;i<n*100000;i++);
}
```


​    
​    
​    

注意使用io函数写需要弄清楚每个设备接口的基地址 在使用pc端查看terminal端口的打印结果时，选择的板间通信接口应该是标准rs232接口，否则在界面上看不到输出的结果
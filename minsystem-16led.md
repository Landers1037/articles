---
title: 基于最小系统—16位led流水灯
name: minsystem-16led
date: 2018-05-25
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


### 基于Xilinx平台

### 仅提供实验代码和引脚约束

实验代码

```verilog
#include "xparameters.h"
#include "xtmrctr.h"
#include "xintc.h"
#include "xil_exception.h"

#define TMRCTR_DEVICE_ID	XPAR_TMRCTR_0_DEVICE_ID
#define INTC_DEVICE_ID		XPAR_INTC_0_DEVICE_ID
#define TMRCTR_INTERRUPT_ID	XPAR_INTC_0_TMRCTR_0_VEC_ID
#define TIMER_CNTR_0	0
#define RESET_VALUE 0X5F5E100

void TimerCounterHandler(void *CallBackRef,u8 TmrCtrNumber);
XIntc InterruptController;
XTmrCtr TimerCounterInst;
u32 LedBits;
int main(void)
{
	int Status;
	LedBits=0;
	Xil_Out32(XPAR_LED_16BITS_BASEADDR+0X4,0X0);
	Status=XTmrCtr_Initialize(&TimerCounterInst,XPAR_TMRCTR_0_DEVICE_ID);
		if(Status !=XST_SUCCESS){
		return XST_FAILURE;
	}
	Status=XIntc_Initialize(&InterruptController,INTC_DEVICE_ID);
		if(Status !=XST_SUCCESS){
			return XST_FAILURE;
	}
	Status=XIntc_Connect(&InterruptController,TMRCTR_INTERRUPT_ID,
					(XInterruptHandler)XTmrCtr_InterruptHandler,
					(void *)&TimerCounterInst);
		if(Status !=XST_SUCCESS){
			return XST_FAILURE;
	}
	Status=XIntc_Start(&InterruptController, XIN_REAL_MODE);
		if(Status !=XST_SUCCESS){
			return XST_FAILURE;
	}
	XIntc_Enable(&InterruptController,TMRCTR_INTERRUPT_ID);
	microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,&InterruptController);
	microblaze_enable_interrupts();
	XTmrCtr_SetHandler(&TimerCounterInst,TimerCounterHandler,
			&TimerCounterInst);
	XTmrCtr_SetOptions(&TimerCounterInst,TIMER_CNTR_0,
				XTC_INT_MODE_OPTION | XTC_AUTO_RELOAD_OPTION | XTC_DOWN_COUNT_OPTION);
	XTmrCtr_SetResetValue(&TimerCounterInst,TIMER_CNTR_0,RESET_VALUE);
	XTmrCtr_Start(&TimerCounterInst,TIMER_CNTR_0);
	while(1);
	return XST_SUCCESS;
}

void TimerCounterHandler(void *CallBackRef,u8 TmrCtrNumber)
{
	Xil_Out32(XPAR_LED_16BITS_BASEADDR,1<<LedBits);
	LedBits++;
	if(LedBits==16)
		LedBits=0;
}
```


​    

引脚约束


```xml
NET "LED_16Bits_GPIO_IO_O_pin<0>"		LOC = "T8" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<1>"		LOC = "V9" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<2>"		LOC = "R8" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<3>"		LOC = "T6" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<4>"		LOC = "T5" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<5>"		LOC = "T4" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<6>"		LOC = "U7" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<7>"		LOC = "U6" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<8>"		LOC = "V4" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<9>"		LOC = "U3" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<10>"		LOC = "V1" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<11>"		LOC = "R1" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<12>"		LOC = "P5" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<13>"		LOC = "U1" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<14>"		LOC = "R2" | IOSTANDARD="LVCMOS33";
NET "LED_16Bits_GPIO_IO_O_pin<15>"		LOC = "P2" | IOSTANDARD="LVCMOS33";
```

感谢wzx大佬的贡献
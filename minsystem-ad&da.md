---
title: 基于最小系统——AD&DA转换
name: minsystem-ad&da
date: 2018-06-21
id: 0
tags: [学习笔记,最小系统]
categories: []
abstract: ""
---


AD
==

```verilog
/*
 * AD.c
 *
 *  Created on: 2018-5-31
 *      Author: 郭
 */
#include "xparameters.h"
#include "xspi.h"
#include "xintc.h"
#include "xil_exception.h"

#define BUFFER_SIZE  2
void SpiIntrandler(void *CallBackRef,u32 Statusevent,u32 Bytecount);
static XIntc IntcInstance;
static XSpi SpiInstance;
volatile int TransferInProgress;
int Error;
u8 ReadBuffer[BUFFER_SIZE];
u8 WriteBuffer[BUFFER_SIZE];
int main(void)
{
int  Status;
Status=XSpi_Initialize(&SpiInstance,XPAR_SPI_0_DEVICE_ID);
Status=XIntc_Initialize(&IntcInstance,XPAR_INTC_0_DEVICE_ID);
Status=XIntc_Connect(&IntcInstance,XPAR_INTC_0_SPI_0_VEC_ID,(XInterruptHandler) XSpi_InterruptHandler,(void *)&SpiInstance);
Status=XIntc_Start(&IntcInstance,XIN_REAL_MODE);
XIntc_Enable(&IntcInstance,XPAR_INTC_0_SPI_0_VEC_ID);
microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,&IntcInstance);
microblaze_enable_interrupts();
XSpi_SetStatusHandler(&SpiInstance,&SpiInstance,(XSpi_StatusHandler)SpiIntrandler);
Status = XSpi_SetOptions(&SpiInstance,XSP_MASTER_OPTION |XSP_CLK_PHASE_1_OPTION);
Status = XSpi_SetSlaveSelect(&SpiInstance,1);
XSpi_Start(&SpiInstance);
while(1){
TransferInProgress = TRUE;
XSpi_Transfer(&SpiInstance,WriteBuffer,ReadBuffer,2);
while(TransferInProgress);
u16  temp;

temp = ReadBuffer[1] << 8;
temp += ReadBuffer[0];

xil_printf("adc = %d\n\r", temp);
int i;
for(i = 0; i < 5000000; i++);
}
return XST_SUCCESS;
}

void SpiIntrandler(void *CallBackRef, u32 StatusEvent, u32 ByteCount)
{
    TransferInProgress = FALSE;
    if  (StatusEvent != XSP_SR_RX_EMPTY_MASK){
    Error++;
}
     }
```


​    
​    
​    

DA
==

```verilog
#include "xparameters.h"
#include "xspi.h"
#include "xintc.h"
#include "xil_exception.h"

#define BUFFER_SIZE  2
void SpiIntrandler(void *CallBackRef,u32 Statusevent,u32 Bytecount);
static XIntc IntcInstance;
static XSpi SpiInstance;
volatile int TransferInProgress;
int Error;
u8 ReadBuffer[BUFFER_SIZE];
u8 WriteBuffer[BUFFER_SIZE];
int main(void)
{
int  Status,Count;
Status=XSpi_Initialize(&SpiInstance,XPAR_SPI_0_DEVICE_ID);
Status=XIntc_Initialize(&IntcInstance,XPAR_INTC_0_DEVICE_ID);
Status=XIntc_Connect(&IntcInstance,XPAR_INTC_0_SPI_0_VEC_ID,(XInterruptHandler) XSpi_InterruptHandler,(void *)&SpiInstance);
Status=XIntc_Start(&IntcInstance,XIN_REAL_MODE);
XIntc_Enable(&IntcInstance,XPAR_INTC_0_SPI_0_VEC_ID);
microblaze_register_handler((XInterruptHandler)XIntc_InterruptHandler,&IntcInstance);
microblaze_enable_interrupts();
XSpi_SetStatusHandler(&SpiInstance,&SpiInstance,(XSpi_StatusHandler)SpiIntrandler);
Status = XSpi_SetOptions(&SpiInstance,XSP_MASTER_OPTION |XSP_CLK_PHASE_1_OPTION);
Status = XSpi_SetSlaveSelect(&SpiInstance,1);
XSpi_Start(&SpiInstance);
while(1){
WriteBuffer[0]=(u8)(Count);
WriteBuffer[1]=(u8)(Count>>8)&0x0f;
Count=Count+43;
if(Count==4096)
	Count=0;
TransferInProgress = TRUE;
XSpi_Transfer(&SpiInstance,WriteBuffer,(void*)0,2);
while(TransferInProgress);
}
return XST_SUCCESS;
}
void SpiIntrandler(void *CallBackRef, u32 StatusEvent, u32 ByteCount)
{
    TransferInProgress = FALSE;
    if  (StatusEvent != XST_SPI_TRANSFER_DONE){
    Error++;
}
     }
```


​    
​    
​    

感谢GJL大佬的分享
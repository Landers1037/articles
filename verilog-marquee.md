---
title: verilog实现流水灯
name: verilog-marquee
date: 2018-03-26
id: 0
tags: [verilog]
categories: []
abstract: ""
---


```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date:    16:45:44 08/17/2012 
// Design Name: 
// Module Name:    led 
// Project Name: 
// Target Devices: 
// Tool versions: 
// Description: 
//
// Dependencies: 
//
// Revision: 
// Revision 0.01 - File Created
// Additional Comments: 
//
//////////////////////////////////////////////////////////////////////////////////

module led(
    input clk,
	 input reset,
	 output [3:0] led_out
	 );
   reg[26:0]counter;
	always@(posedge clk)
	begin
	 if(reset)
	    counter<=0;
		 else
		 counter<=counter+1;
		 end
	
		assign led_out=counter[26:23];
```


​    
    endmodule

流水灯打包 [https://drive.google.com/open?id=1kP50XV2VeEsLHB9JCXDWeDa8PAxXGW7V](https://drive.google.com/open?id=1kP50XV2VeEsLHB9JCXDWeDa8PAxXGW7V)
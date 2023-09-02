---
title: verilog实现2选1数据选择器
name: verilog-chooser
date: 2018-03-27
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---


**实验代码**

```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date:    17:55:55 12/10/2017 
// Design Name: 
// Module Name:    mux2to1 
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
module mux2to1(d0,d1,s,q  );
	input d0,d1,s;
	output reg q;
	
	always@(d0,d1,s)begin
	if(s==1) q<=d1;
	else q<=d0;
	end
```


​    	 

    endmodule
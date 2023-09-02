---
title: verilog实现步进电机
name: verilog-stepper-motor
date: 2018-03-26
id: 0
tags: [verilog]
categories: []
abstract: ""
---


感谢某位姓王大佬赞助<!--more-->

```verilog
`timescale 1ns / 1ps

module topblock(CLK_50M,A,CLR1,CLR2,state);
	input CLK_50M,A,CLR1,CLR2;
	output [2:0] state;
	wire CP;
	
	Divider50MHz U0(CLR1,CLK_50M,CP);
	steppermotor U1(A,CP,CLR2,state);
endmodule

module Divider50MHz(CLR,CLK_50M,CLK_1HzOut); 
	input CLR,CLK_50M;
	output reg CLK_1HzOut;
	reg [24:0] Count_DIV;
	//parameter CLK_Frq = 50000000,OUT_Frq = 1;
	parameter CLK_Frq = 50000000,OUT_Frq = 5000000;
	
	always @(posedge CLK_50M,negedge CLR)
	begin 
		if(!CLR) begin CLK_1HzOut<=0;Count_DIV<=0;end 
		else begin
			if(Count_DIV<(CLK_Frq/(2*OUT_Frq)-1)) Count_DIV<=Count_DIV+1'b1;
			else begin Count_DIV<=0;CLK_1HzOut<=~CLK_1HzOut;end
		end
	end
endmodule
		
module steppermotor(A,CP,CLR,state);
	input A,CP,CLR;
	output reg [2:0] state;
	
	parameter S0=3'b111,S1=3'b000,S2=3'b110,S3=3'b010;
	parameter S4=3'b011,S5=3'b001,S6=3'b101,S7=3'b100;
	
	always @(posedge CP,negedge CLR)
	begin
		if(~CLR) state<=S2;
		else case(state)
			S0:state<=S1;
			S1:state<=S2;
			S2:begin state<=(A==0)?S3:S7;end
			S3:begin state<=(A==0)?S4:S2;end
			S4:begin state<=(A==0)?S5:S3;end
			S5:begin state<=(A==0)?S6:S4;end
			S6:begin state<=(A==0)?S7:S5;end
			S7:begin state<=(A==0)?S2:S6;end
			endcase
	end
			
endmodule
```
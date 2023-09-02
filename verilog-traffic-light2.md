---
title: verilog实现交通灯（2）
name: verilog-traffic-light2
date: 2018-03-27
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---


```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date:    17:06:14 12/26/2017 
// Design Name: 
// Module Name:    traffic 
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
module traffic (CP,red,green,yellow);
    input CP;
	 output reg [1:0]red;
	 output reg [1:0]green;
	 output reg [1:0]yellow;
	 reg [3:0]state;
	 reg [5:0]cnt;
	 always@(posedge CP )begin
			cnt<=0;
			state<=4'b0000;
		   case(state)
			  4'b0000:begin
			      red<=2'b11;
			      green<=2'b11;
			      yellow<=2'b11;
					state<=4'b0001;
					end
			  4'b0001:
			      if(cnt==30)begin
					   state<=4'b0010;
						cnt<=0;
						end
					else begin
					   cnt<=cnt+1'b1;
			         red<=2'b10;
			         green<=2'b01;
			         yellow<=2'b00;
						state<=4'b0001;
						end
			   4'b0010:
				    if(cnt==5)begin
					   state<=4'b0011;
						cnt<=0;
						end
					 else begin
					   cnt<=cnt+1'b1;
						red<=2'b10;
			         green<=2'b00;
			         yellow<=2'b01;
						state<=4'b0010;
						end
				4'b0011:
				    if(cnt==20)begin
					    state<=4'b0100;
						 cnt<=0;
						 end
					 else begin
					    cnt<=cnt+1'b1;
						 red<=2'b01;
			          green<=2'b10;
			          yellow<=2'b00;
						 state<=4'b0011;
						 end
				4'b0101:
				    if(cnt==5)begin
					    state<=4'b0001;
						 cnt<=0;
						 end
					 else begin
					    cnt<=cnt+1'b1;
						 red<=2'b01;
			          green<=2'b00;
			          yellow<=2'b10;
						 state<=4'b0101;
						 end
				 endcase
	  end
endmodule
```


​    

测试代码


```verilog
module test;

	// Inputs
	reg CP;

	// Outputs
	wire [1:0] red;
	wire [1:0] green;
	wire [1:0] yellow;

	// Instantiate the Unit Under Test (UUT)
	traffic uut (
		.CP(CP), 
		.red(red), 
		.green(green), 
		.yellow(yellow)
	);
always begin
 CP=1'b0;
 #5;
 CP=1'b1;
 #5;
 end
	initial begin
		// Initialize Inputs
		CP = 0;
		#100;
```


​    		
```verilog
	end
      
endmodule
```


交通灯打包下载 [https://drive.google.com/open?id=1ZoUkJqYmHhWgpwRTWb-Q7oLoLSs-Vq7q](https://drive.google.com/open?id=1ZoUkJqYmHhWgpwRTWb-Q7oLoLSs-Vq7q)
---
title: verilog实现D触发器
name: verilog-d-trigger
date: 2018-03-27
id: 0
tags: [verilog]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date:    20:05:15 12/11/2017 
// Design Name: 
// Module Name:    triger 
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
module Dtriger(d,clk,reset,set,q,qf);
    input [1:0] d;
    input clk;
    input reset;
    input set;
    output reg[1:0] q;//输出端q
    output reg[1:0] qf;//反向输出q非
    
       always @ (posedge clk) begin          //时钟上升沿时，触发D触发器
          
                     if(!set&& reset)       //若set为低电平，reset为高电平时，D触发器置1
                            begin
                                   q<= 2'b11 ;
                                   qf<= 2'b00 ;
                            end
                     else if (set&& !reset)    //若set为高电平，reset为低电平时，D触发器置0
                            begin
                                   q<= 2'b00 ;
                                   qf<= 2'b11 ;
                            end
                     else                     //D触发器正常工作
                            begin
                                   q<= d ;
                                   qf<= ~d ;
                            end
              end
endmodule
```
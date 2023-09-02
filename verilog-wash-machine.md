---
title: verilog实现自动洗衣机
name: verilog-wash-machine
date: 2018-03-21
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---


造福大家，分享给以后需要写的伙伴
实验流程设计：

1、系统功能 基本功能：待机5s正转60s待机5s反转60s四种基本状态，能够实现设置循环次数后待机，并显示循环次数，遇到紧急情况可按紧急按钮暂停洗衣机的工作，循环次数到0自动报警。<!--more-->

 2、系统设计
 ①分频模块：将系统给的50MHz的频率通过分频模块变成1Hz的clk。
 ②三个七段数码管，分别表示倒计时的个位tia，十位tib和循环剩余次数tim。
 ③洗衣机主程序： 控制模块采用有限状态机实现对洗衣机工作状态的控制。启动start后，控制器首先进入待机s0状态，时间从5秒倒计时，如果没有到0秒则继续等待，时间自减；当t=0,进入洗衣机正转s1状态，时间从60秒倒计时，如果没有到0秒则继续等待，时间自减；当t=0, 进入洗衣机待机s2状态，同理等待5秒；当t为零后，进入s3洗衣机反转状态，时间从60秒倒计时，如果没有到0秒则继续等待，时间自减；整个过程依次循环。紧急状态stop，当按下紧急停止按钮后，处于正反转状态中的电机停止转动，同时紧急报警 LED灯亮，因此紧急状态不设定独立状态。
 ④在实验过程中发现不能在数码管上同时显示出十位和个位，于是采用四位动态扫描模块，分频400hz分时显示数码管，因为人眼的视觉暂留效应所以看上去像是三个数码管同时亮了


```verilog
module Divider50MHZ(CR,clk,clk0);
  input CR,clk;
  output reg clk0;
  reg [24:0] Count_DIV;
  always @(posedge clk,negedge CR)
  if(!CR) begin
    clk0<=0;
    Count_DIV<=0;
  end
  else begin
    if(Count_DIV<4999999)
      Count_DIV<=Count_DIV+1'b1;
  else begin
    Count_DIV<=0;
    clk0 <= ~clk0;
  end
end
endmodule
```

```verilog
module seg7(clk,a,b,t,D,w); 
input \[3:0\]a,b,t; reg \[3:0\]code; 
input clk; 
output\[6:0\]D; 
output \[2:0\]w; 
reg \[6:0\]D; 
integer clk\_div; 
reg clk\_400hz; 
always @(posedge clk) begin if(clk\_div==100000) 
	begin clk\_div<=1'b0; clk\_400hz<=~clk\_400hz; 
	end else clk\_div<=clk\_div+1'b1; end reg \[2:0\]w=3'b110; always @(posedge clk_400hz) w<={w\[1:0\],w\[2\]}; always @(w) case(w) 3'b110:code=a; 3'b101:code=b; 3'b011:code=t; endcase

always@(code) begin case(code) 
4'b0000: D<=7'b0000001; 
4'b0001: D<=7'b1001111; 
4'b0010: D<=7'b0010010; 
4'b0011: D<=7'b0000110; 
4'b0100: D<=7'b1001100; 
4'b0101: D<=7'b0100100; 
4'b0110: D<=7'b1100000; 
4'b0111: D<=7'b0001111; 
4'b1000: D<=7'b0000000; 
4'b1001: D<=7'b0000100; 
default: D<=7'b1111111; 
endcase 
end 
endmodule

module washing(CR,clk,rst,stop,start,add,fore,back,a,b,alarm,led,t,D,w); input CR,clk,rst,start,add,stop; //stop为紧急状态信号 
wire clk0; output reg fore,back,alarm; //display 
output reg \[3:0\]a,b,t; //a为倒计时个位，b为倒计时十位 
output reg \[2:0\]led; 
output wire \[6:0\]D; 
output \[2:0\]w; 
reg \[3:0\]state; 
reg \[3:0\]count; 
parameter s0=4'b0001,s1=4'b0010,s2=4'b0100,s3=4'b1000; Divider50MHZ part1(CR,clk,clk0); 
always@(posedge add or posedge rst) // 循环次数 count 设置 
begin if(rst) count<=1'b0; else begin if(start==0) begin 			if(count<15) count<=count+1'b1; 
	else count<=1'b0; 
		end 
	end 
end always@(posedge clk0 or posedge rst) begin if(rst) // 系统复位 
begin back<=1'b0; fore<=1'b0; a<=4'b0000; b<=4'b0000; state<=s0; led<=3'b001; alarm=1'b0; 
	end 
else begin if(!stop&& start) //start为1，stop为0则正常工作 
begin alarm=0; 
if(t) // 循环次数 t>0, 开始工作 
begin case(state) s0:begin if(b==0 && a==0) 
//时间为0，状态转移 
begin state<=s1; a<=4'b0100;
//4 b<=4'b0000;//0 
led<=3'b001; back<=1'b0; fore<=1'b0; end else begin state<=s0; led<=3'b100; back<=1'b0; fore<=1'b0; end end s1:begin if(b==0&&a==0) begin state<=s2; a<=4'b1001;//9 
b<=4'b0001;//5 
back<=1'b0; 
fore<=1'b1; 
led<=3'b010; 
end 
else begin state<=s1; led<=3'b001; back<=1'b0; fore<=1'b0; end end 
s2:begin if(b==0&&a==0) 
begin state<=s3; a<=4'b0100;//4 
b<=4'b0000;//0 
back<=1'b0; fore<=1'b0; led<=3'b001; end else begin state<=s2; back<=1'b0; fore<=1'b0; led<=3'b010; end end s3:begin if(b==0&&a==0) begin state<=s0; a<=4'b1001;//4 
b<=4'b0001;//0 
back<=1'b1; fore<=1'b0; led<=3'b100; end else begin state<=s3; back<=1'b0; fore<=1'b0; led<=3'b001; end end default:state<=s0; endcase if({b,a}>0) // 倒计时控制部分 
begin if(a==0) //a为0,则a赋值9，b自减1 
begin a<=9;//9 
b<=b-1'b1; end else a<=a-1'b1; end else if(a==0&&b==0&&state==s0&&!stop&&led==3'b100) begin // 一次循环结束t自减1 
t<=t-1'b1; 
	end 
end 
else begin back<=1'b0; fore<=1'b0; a<=4'b0000; b<=4'b0000; alarm=0; state<=s0; led<=3'b001; 
	end 
end 
else if(stop && start) //en 为 1，进入紧急状态 
begin back<=1'b0; fore<=1'b0; alarm=1'b1; 
	end 
else //start为0，给循环次数t赋值 
begin t<=count; a<=0; b<=0; alarm=1'b0; 
		end 
	end 
end 
seg7 part2(clk,a,b,t,D,w); 
endmodule
```

管脚约束

​    

部分代码修改提示 时钟的1hz因为实验要求改为分频2hz 59s的倒计时因为时间过长改为19s
---
title: verilog语言实现10进制计数器
name: verilog-10base-convert
date: 2018-03-26
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---


```verilog
module counter10(clk_50MHZ,seg,an);

	input clk_50MHZ;
	output [6:0]seg;
	output [3:0]an;
	reg  [3:0]out=0;
	//seg7段译码器信号端 an4选择器信号端
```

​    	
<!--more-->


```verilog
module counter10(clk_50MHZ,seg,an);

	input clk_50MHZ;
	output [6:0]seg;
	output [3:0]an;
	reg  [3:0]out=0;
	//seg7段译码器信号端 an4选择器信号端
```

​    	<!--more-->
​    	

```verilog
reg [1:0]clk_for_an=0;
​    	reg [25:0]Count_DIV=0;
​    	reg clk_1HZ=0;
​    	reg [3:0]an=0;
​    	reg [6:0]seg=0;
​    	parameter CLK_Freq=50000000;//定义时钟信号为50mhz

​    	
​    	
​    
​    			
​    			
​    	/*always@(posedge clk_50MHZ)  begin
​    		if(clk_for_an<3)
​    			clk_for_an=clk_for_an+1;
​    		else
​    			clk_for_an=2'b00;
​    	end*/
​    	
​    	always@(posedge clk_50MHZ)  begin
​    	    if(Count_DIV<(CLK_Freq/2-1))
​    		   Count_DIV<=Count_DIV+1'b1;
​    		 else  begin
​    		   Count_DIV<=0;
​    			clk_1HZ=~clk_1HZ;
​    	    end
​    	end//对50mhz进行分频，达到一半频率进行一次分频

​    	
​    	
​    	
​    	always@(posedge clk_50MHZ) begin
​    	   case({Count_DIV[1],Count_DIV[0]})
​    			2'b00:an<=4'b0111;
​    			2'b01:an<=4'b1011;
​    			2'b10:an<=4'b1101;
​    			2'b11:an<=4'b1110;
​    		endcase
​    	end//对an进行赋值

​    	
​    	always@(posedge clk_50MHZ)  begin
​    	  if(Count_DIV==24999999&&clk_1HZ)//当时钟信号为24999999时为有效清零信号
​    			begin 
​    				     if(out==4'b1001)
​    					     out<=4'b0000;
​    					  else
​    					     out<=out+1'b1;
​    			 end
​    	
​    	end//运算加法器部分逢9置零

​    	
​    	always@(posedge clk_50MHZ)  begin
​    		case(out)
​    		0:seg=7'b100_0000;
​    		1:seg=7'b111_1001;
​    		2:seg=7'b010_0100;
​    		3:seg=7'b011_0000;
​    		4:seg=7'b001_1001;
​    		5:seg=7'b001_0010;
​    		6:seg=7'b000_0010;
​    		7:seg=7'b111_1000;
​    		8:seg=7'b000_0000;
​    		9:seg=7'b001_0000;
​    		endcase
​    	end
​    	endmodule
```



---
title: verilog实现数字钟
name: verilog-digital-clock
date: 2018-03-31
id: 0
tags: [verilog]
categories: []
abstract: ""
---


verilog实现数字钟，要求功能实现12小时到24小时进制转换，校时校分功能，闹钟，整点报时功能 **顶层模块**
<!--more-->


verilog实现数字钟，要求功能实现12小时到24小时进制转换，校时校分功能，闹钟，整点报时功能 **顶层模块**<!--more-->

```verilog
module top(
    input clk,//50MHz默认时钟信号
    input adj,//按键1，实现校时功能，2位宽，共控制4种循环状态（正常显示，闪烁校时，闪烁校分，显秒）
	 input alarm,//按键2，实现闹钟功能，2位宽，共控制4种循环状态（正常显示，设置闹钟时，设置闹钟分，显示闹钟开关状态）
	 input sw,//按键3，实现进制切换功能，1位宽，共控制2种循环状态（正常显示，12进制显示）
	 input en,//按键4，作为enter键，1位宽，在校时和设置闹钟模式下用作增计数，显秒时用于秒清零，还用于开关闹钟
    output [0:6] seg,//七段数码管显示输出
    output [0:3] sc,//扫描四个数码管输出
	 output led_alarm,led_s,led_chime,led_am//闹钟长亮灯,秒钟闪烁灯，报时闪烁灯，闹钟开关长亮灯
    );
	
wire [5:0] ct;//用于存储按键状态，共6位
wire [7:0] sw_Hour,hour_al,minute_al;//中间变量，用于显示进制切换后的小时，和设置闹钟时的小时和分钟
wire [7:0] Hour,Minute,Second;//中间变量，时，分，秒的两位数字各占4位宽（0-9）

supply0 rst;//除实现秒清零功能外，该输入端其余时候均接地
			
counter main(clk,rst,{ct[5],ct[3:2]},Hour,Minute,Second,led_s);
//主函数为时分秒的计数器，输入为adj与enter键，中间输出为时分秒的计数，终端输出为秒闪烁灯和报时长亮灯

display dis(clk,ct[4:0],alarm_state,en,sw_Hour,Minute,Second,hour_al,minute_al,seg,sc);
//显示函数，输入为adj,alarm，sw和en键，中间输入有闹钟开关状态，进制转换后的小时计数，分钟计数，秒钟计数，设置闹钟状态下的小时和分钟，输出为4个七段数码管

controller c0(clk,rst,adj,alarm,sw,en,ct);
//控制函数，输入为默认时钟信号，秒清零信号，四个按键，输出为6位状态编码

hour_switch h0(clk,Hour,ct[4],sw_Hour,led_am);
//时钟进制切换函数，输入为计数器的时钟，sw按键，输出为进制转换以后的小时和进制转换后显示上午的长亮灯
//在正常显示24进制时，display函数的小时计数输入信号仍然来自于hour_switch函数的输出sw_Hour，而非counter函数的输出Hour

alarm al0(clk,rst,{ct[5],ct[1:0]},Hour,Minute,hour_al,minute_al,led_alarm,alarm_state);
//闹钟设置函数，输入为alarm与enter键，中间输入为counter函数的输出Hour,Minute，中间输出为设置闹钟状态下的小时和分钟（将传递给dispay来显示），
//终端输出为闹钟长亮灯和闹钟开关状态

endmodule
```


**60进制进位**

```verilog
module counter_60(
    input clk,
    input rst,
	 input en,//由上一级传递来的进位信号
    output reg[3:0] cnth=0,//高四位输出
    output reg[3:0] cntl=0,//低四位输出
	 output co
    );//60进制计数器

assign co = cnth==4'd5&&cntl==4'd9;//在第59次计数时产生对下一级的进位输出信号
always@(posedge clk or posedge rst)
begin
	if(rst){cnth,cntl}<=8'd0;//异步清零，仅秒计数时使用
	else if(en)//上一级产生了进位信号
		if(cnth==5&&cntl==9){cnth,cntl}<=8'd0;//若计数达到59，则归零
		else if(cntl==4'd9)
		begin
			cntl<=4'd0;
			cnth<=cnth+4'd1;//若低位为9，则低位归零，高位加1
		end 
		else cntl<=cntl+4'd1;//若低位不为9，则低位加1
	else 
		begin
			cnth<=cnth;
			cntl<=cntl;//上一级未产生进位信号时，则维持高低位的输出不变
		end

end

endmodule
```


**24进制进位**

```verilog
module counter_24(
    input clk,
    input rst,
	 input en,
    output reg[3:0] cnth=0,
    output reg[3:0] cntl=0,
	 output co
    );//24进制计数器

assign co = cnth==4'd2&&cntl==4'd3;
always@(posedge clk or posedge rst)
begin
	if(rst){cnth,cntl}<=8'd0;
	else if(en)
		if(cnth==2&&cntl==3){cnth,cntl}<=8'd0;//若计数达到23，则归零
		else if(cntl==4'd9)
		begin
			cntl<=4'd0;
			cnth<=cnth+4'd1;
		end
		else cntl<=cntl+4'd1;
	else 
		begin
			cnth<=cnth;
			cntl<=cntl;
		end

end

endmodule
```


**6进制进位**

```verilog
module counter_6(
    input clk,
    input rst,
    input en,
    output co
    );
	 
reg [2:0] cnt = 0;
assign co = cnt == 5;
```


​    
​    
```verilog
always@(posedge clk or posedge rst)
begin
	if(rst)cnt=0;
	else if(en) begin
		if(cnt==5)cnt=0;
		else cnt = cnt+1;
		end
	else cnt = cnt;
end
endmodule
```


**比较进位判断**

```verilog
module compare(
	 input clk,
    input [7:0] Hour,
    input [7:0] Minute,
	 input [7:0] Second,
    input [7:0] hour_al,
    input [7:0] minute_al,
    output led_alarm
    );

reg [5:0] cnt=0;
reg [4:0] num=0;
wire [5:0] dou_num;
assign dou_num = num + num;
assign led_alarm = (cnt < dou_num) & cnt[0];

divider50Mhz U2(clk,rst,CP_2Hz);//秒计数
defparam U0.N = 5,
			U0.clk_freq = 6,
			U0.out_freq = 2;//改为1Hz(

always@(Hour)
begin
	if(Hour[7:4]==0) num = Hour[3:0];
	else if(Hour[7:4]==1) num = Hour[3:0] + 5'b01010;
	else if(Hour[7:4]==2) num = Hour[3:0] + 5'b10100;
end

always@(posedge CP_2Hz)
begin
	if(Hour==hour_al & Minute==minute_al) begin
		if(Second==0) cnt <= 0;
		else cnt <= cnt + 1;
	end
end
endmodule
```


**12-24小时转换**

```verilog
module hour_switch(
	 input clk,
    input [7:0] hour_in,
    input en,//12进制显示使能信号,即sw键
    output reg [7:0] hour_out,
	 output led_am
    );
	 
	 assign led_am = en&(hour_in[7:4]==0|hour_in[7:4]==1&hour_in[3:0]<2);//进制切换后上午时间该灯长亮

always@(posedge clk)
	begin
		if(en) begin//长按sw键则显示进制切换后的时间，松开则正常显示
			case(hour_in)
				8'b0000_0000: hour_out <= 8'b0001_0010;
				8'b0001_0011: hour_out <= 8'b0000_0001;
				8'b0001_0100: hour_out <= 8'b0000_0010;
				8'b0001_0101: hour_out <= 8'b0000_0011;
				8'b0001_0110: hour_out <= 8'b0000_0100;
				8'b0001_0111: hour_out <= 8'b0000_0101;
				8'b0001_1000: hour_out <= 8'b0000_0110;
				8'b0001_1001: hour_out <= 8'b0000_0111;
				8'b0010_0000: hour_out <= 8'b0000_1000;
				8'b0010_0001: hour_out <= 8'b0000_1001;
				8'b0010_0010: hour_out <= 8'b0001_0000;
				8'b0010_0011: hour_out <= 8'b0001_0001;
				default: hour_out <= hour_in;//1am~12am/pm与正常显示时相同
			endcase
		end
		else hour_out <= hour_in;//若无进制切换使能信号，则输出counter的输出信号以正常显示
	end
endmodule
```

**控制器**

```verilog
module controller(
	 input clk,
	 input rst,
    input adj,
	 input alarm,
	 input sw,
	 input en,
    output [5:0] ct
    );
reg sw_cnt=0;//1位进制转换开关
reg [1:0] adj_cnt=0,alarm_cnt=0;//校时键adj与闹钟设置键alarm的按键状态存储器，各存4个状态
wire en_clk;//按键扫描频率，7Hz时钟信号

assign en_adj = {alarm_cnt,sw,en} == 4'b0000;//用于adj按键使能，保证其余3个按键均无动作
assign en_alarm = {adj_cnt,sw,en} == 4'b0000;//用于alarm按键使能，保证其余3个按键均无动作
assign en_sw = {adj_cnt,alarm_cnt,en} == 5'b00000;//用于sw按键使能，保证其余3个按键均无动作
assign ct = {en&en_clk,sw_cnt,adj_cnt,alarm_cnt};//第5位为en&en_clk，表示按下en键且7Hz扫描信号为高电平；第4位为sw，第[3:2]位为adj，第[1:0]位为alarm

divider U0(clk,rst,en_clk);
defparam U0.N = 25,
			U0.clk_freq=50_000_000,
			U0.out_freq=7;//仿真用700_000
//分频得到7Hz扫描时钟信号			

always@(posedge clk)
	begin
		if(en_clk) begin//每0.14s扫描一次按键，符合人的短按持续时间；若长按，则每秒扫描7次
			if(adj&en_adj) adj_cnt <= adj_cnt + 2'b01;//adj键按下且符合使能条件时，状态+1
			else if(alarm&en_alarm) alarm_cnt <= alarm_cnt + 2'b01;//alarm键按下且符合使能条件时，状态+1
			else if(sw&en_sw) sw_cnt = ~sw_cnt;//sw键按下且符合使能条件时，状态+1
			else ;
		end
	end
```


​    
​    endmodule


**译码器转换**

```verilog
module decode(
	 input clk,
    input [3:0] num,
    output reg[6:0] seg//a,b,c,d,e,f,g
    );

always@(posedge clk)
begin
	case(num)
		4'h0: seg = 7'b0000001;   //0
		4'h1: seg = 7'b1001111;   //1
		4'h2: seg = 7'b0010010;   //2
		4'h3: seg = 7'b0000110;   //3
		4'h4: seg = 7'b1001100;   //4
		4'h5: seg = 7'b0100100;   //5
		4'h6: seg = 7'b0100000;   //6
		4'h7: seg = 7'b0001111;   //7
		4'h8: seg = 7'b0000000;   //8
		4'h9: seg = 7'b0000100;   //9
		4'hf: seg = 7'b1111111;	  //全灭，用于闪烁
		default: seg = 7'b0000001; 
	endcase
end

endmodule
```


**数码管显示**

```verilog
module display(
    input clk,
    input [4:0] ct,
	 input alarm_state,
	 input en,
    input [7:0] Hour,
    input [7:0] Minute,
    input [7:0] Second,
    input [7:0] hour_alarm,
    input [7:0] minute_alarm,
    output [6:0] seg,
    output [3:0] sc
    );

wire [3:0] num;//4位宽用于存放数码管所显示的数字

scan s0(clk,ct,alarm_state,en,Hour,Minute,Second,hour_alarm,minute_alarm,num,sc);
//用于扫描数码管，控制其显示；由于扫描速率设为了200Hz，故人眼无法识别是否闪烁
decode dec0(clk,num,seg);
//用于每个数码管的显示译码

endmodule
```

**动态扫描模块**

```verilog
module scan(
    input clk,
	 input [4:0] ct,
	 input alarm_state,
	 input en,
    input [7:0] Hour,Minute,Second,
	 input [7:0] hour_al,minute_al,
    output reg [3:0] num,
    output reg [3:0] scan
    );

parameter N=6;//用于调整闪烁速率
reg [N-1:0] cnt=0;//数码管闪烁计数器，6位宽可存放0~64
wire en_200Hz;

divider U3(clk,1'b0,en_200Hz);
defparam U3.N=25,
			U3.clk_freq = 50_000_000,
			U3.out_freq = 200;//仿真用20_000_000
//分频得到200Hz的时钟信号

assign set_h = ct[3:2] == 2'b01;//校时
assign set_m = ct[3:2] == 2'b10;//校分
assign set_s = ct[3:2] == 2'b11;//校秒
assign alarm_h = ct[1:0] == 2'b01;//设置闹钟时
assign alarm_m = ct[1:0] == 2'b10;//设置闹钟分
//上述每个状态下对应的两位数码管均会闪烁
assign alarm_sw = ct[1:0] == 2'b11;//设置闹钟开关
assign dis_s = en&ct[3:0]==0;//正常显示时长按且仅按en键用于显秒

assign en_hour = (set_h|alarm_h)&cnt[N-1];//小时闪烁
assign en_minute = (set_m|alarm_m|set_s)&cnt[N-1];//分钟闪烁（秒闪烁）
//cnt的最高位为1时点亮，每秒从0~64循环扫描，共计200次，故闪烁次数每秒约3次

always @ (posedge clk)
	begin
		if(en_200Hz) begin
		cnt <= cnt+1'b1;
		case (cnt[1:0])//由低两位的四种状态分别控制四个数码管，使显示时间平均
		  2'b00: 
			begin 
				scan <= 4'b1110;//最低位数码管显示
				if(en_minute) num <= 4'b1111;//校分，闹钟设置分，秒清零状态下控制最低位闪烁
				else if(alarm_sw) num <= alarm_state;//在设置闹钟开关状态下，最低位显示0或1
				else if(alarm_m|alarm_h) num <= minute_al[3:0];//在闹钟设置小时或分钟的状态下，分钟的低位显示在最低位数码管
				else if(set_s|dis_s) num <= Second[3:0];//在校秒或显秒的状态下，秒的低位显示在最低位数码管
				else num <= Minute[3:0];//其余情况正常显示分钟的低位
			end
		  2'b01:
			begin
				scan <= 4'b1101;//次低位数码管显示    
				if(en_minute|alarm_sw)num <= 4'b1111;//校分，闹钟设置分，秒清零状态下控制次低位闪烁，显示闹钟开关状态时此位不点亮
				else if(alarm_m|alarm_h) num <= minute_al[7:4];//在闹钟设置小时或分钟的状态下，分钟的高位显示在次低位数码管
				else if(set_s|dis_s) num <= Second[7:4];//在校秒或显秒的状态下，秒的高位显示在次低位数码管
				else num <= Minute[7:4];//其余情况正常显示分钟的高位
			end
		  2'b10:
			begin
				scan <= 4'b1011;//次高位数码管显示    
				if(en_hour|set_s|dis_s|alarm_sw) num <= 4'b1111;//校时，设置闹钟小时两种状态下控制闪烁；校秒，显秒和显示闹钟开关状态下此位不点亮
				else if(alarm_m|alarm_h) num <= hour_al[3:0];//在闹钟设置小时或分钟的状态下，小时的低位显示在次高位数码管
				else num <= Hour[3:0];//其余情况正常显示小时的低位
			end
		  2'b11:
			begin
				scan <= 4'b0111;//最高位数码管显示   
				if(en_hour|set_s|dis_s|alarm_sw) num <= 4'b1111;//校时，设置闹钟小时两种状态下控制闪烁；校秒，显秒和显示闹钟开关状态下此位不点亮
				else if(alarm_m|alarm_h) num <= hour_al[7:4];//在闹钟设置小时或分钟的状态下，小时的高位显示在最高位数码管
				else num <= Hour[7:4];//其余情况正常显示小时的高位
			end
		endcase
	end
	end

endmodule
```

**分频器**

```verilog
module divider(
    input clk,
    input rst,
    output en_out
    );//分频器
parameter N=26;
parameter clk_freq = 2;
parameter out_freq = 1;

reg [N-1:0] count;//内部节点，存放计数器的输出值

assign en_out = count == clk_freq/out_freq - 1;//分频输出，仅在counter达到计数的最大值时有输出，且该输出仅持续原始clk信号一个脉冲的时间

always@(posedge clk or posedge rst)
begin
	if(rst)
		begin
			count<=0;//仿真时用于异步清零
		end
	else 
		begin
			if(count<(clk_freq/out_freq - 1 )) count<=count+1;//分频计数器增1计数
			else 
				begin
					count<=0;//分频计数器达到计数的最大值时被清零，此时输出信号产生高电平
				end
		end
end
```


​    
​    endmodule


**闹钟**

```verilog
module alarm(
    input clk,
	 input rst,
    input [2:0] ct,
    input [7:0] Hour,
    input [7:0] Minute,
    output [7:0] hour_al,
    output [7:0] minute_al,
    output led_alarm,
	 output alarm_state//闹钟开关的状态需传递至display函数在数码管上用0或1显示
    );

reg alarm_sw_cnt=0;//闹钟开关标志，初始时为0
assign alarm_state =alarm_sw_cnt;
assign en_hou = ct == 3'b101;//设置闹钟小时状态下en键被7Hz时钟信号扫描到按下时，小时进位，进位信号传递至24进制计数器
assign en_min = ct == 3'b110;//设置闹钟分钟状态下en键被7Hz时钟信号扫描到按下时，分钟进位，进位信号传递至60进制计数器
assign en_sw = ct == 3'b111;//设置闹钟开关状态下en键被7Hz时钟信号扫描到按下时，控制闹钟开关状态改变的en_sw被使能

assign led_alarm = (Hour==hour_al)&(Minute==minute_al)&(alarm_sw_cnt);//比较当前时分与设置的闹钟时分，当相等且在闹钟开关标志为1时，闹钟灯长亮1min

always@(posedge clk) begin
	if(en_sw) alarm_sw_cnt = ~alarm_sw_cnt;//改变闹钟开关状态
	end
```


​    
```verilog
counter_60 min(clk,rst,en_min,minute_al[7:4],minute_al[3:0],);
counter_24 hou(clk,rst,en_hou,hour_al[7:4],hour_al[3:0],);
//设置闹钟状态下计数器的向后进位输出不使用

endmodule
```

 


建立顶层top文件后分别做24小时和12小时的进位仿真 test代码如下  

```verilog
module counter_test;

// Inputs
reg clk;
reg [2:0] ct;

// Outputs
wire [7:0] Hour;
wire [7:0] Minute;
wire [7:0] Second;
wire led_s;
wire led_chime;

// Instantiate the Unit Under Test (UUT)
counter uut (
.clk(clk), 
.ct(ct), 
.Hour(Hour), 
.Minute(Minute), 
.Second(Second), 
.led_s(led_s), 
.led_chime(led_chime)
);
always begin
clk=0;
#5;
clk=1;
#5;
end
initial begin
// Initialize Inputs
clk = 0;
ct = 0;

// Wait 100 ns for global reset to finish
#100;
```


​    
```verilog
// Add stimulus here

end

endmodule

module controller_test;

	// Inputs
	reg clk;
	reg adj;
	reg alarm;
	reg sw;
	reg en;

	// Outputs
	wire [5:0] ct;
	// Instantiate the Unit Under Test (UUT)
	controller uut (
		.clk(clk), 
		.adj(adj), 
		.alarm(alarm), 
		.sw(sw), 
		.en(en),		
		.ct(ct)
	);

	always begin
		clk = ~clk;
		#5;
	end
		
	initial begin
		// Initialize Inputs
		clk = 0;
		adj = 0;
		alarm = 0;
		sw = 0;
		en = 0;

		// Wait 100 ns for global reset to finish
		#10;
		en=1;
		sw=1;
      adj = 1; #10; adj = 0;
		#10;
      adj = 1; #10; adj = 0;
		#10;
      adj = 1; #10; adj = 0;
		#10;
      adj = 1; #10; adj = 0;
		#10;
		
      alarm = 1; #10; alarm = 0;
		#100;
      alarm = 1; #10; alarm = 0;
		#100;
      alarm = 1; #10; alarm = 0;
		#100;
      alarm = 1; #10; alarm = 0;
		#100;
		
      sw = 1; #10; sw = 0;
		#100;
		
      en = 1; #10; en = 0;
		#100;
		
      adj = 1; #10; adj = 0;
		#100;			
		en = 1; #10; en = 0;
		#100;
		
		adj = 1; #10; adj = 0;
		#100;		
		sw = 1; #10; sw = 0;
		#100;			
		alarm = 1; #10; alarm = 0;
		#100;	
		adj = 1; #10; adj = 0;
		#100;		
		adj = 1; #10; adj = 0;
		#100;		

		alarm = 1; #10; alarm = 0;
		#100;		
		adj = 1; #10; adj = 0;
		#100;		
		alarm = 1; #10; alarm = 0;
		#100;		
		alarm = 1; #10; alarm = 0;
		#100;		
		alarm = 1; #10; alarm = 0;
		#100;		
```


​    		
```verilog
		// Add stimulus here

	end
      
endmodule
```


​    

```verilog
module hour_switch_test;

	// Inputs
	reg clk;
	reg [7:0] hour_in;
	reg en;

	// Outputs
	wire [7:0] hour_out;
	wire led_am;
	// Instantiate the Unit Under Test (UUT)
	hour_switch uut (
	   .clk(clk),
		.hour_in(hour_in), 
		.en(en), 
		.hour_out(hour_out),
		.led_am(led_am)
	);
	
	always begin
	clk<=0;
		#5;
		clk<=1;
		#5;
		end
	initial begin
		// Initialize Inputs
		#10;
		clk=0;
      hour_in=0;
		en=0;
```


​    	  
​    
```verilog
		// Wait 100 ns for global reset to finish
		#10;
	
		hour_in = 8'b0010_0011;
		#10;
		hour_in = 8'b0001_0010;
		#10;
		hour_in = 8'b0000_0011;
		#10;
		hour_in = 8'b0001_0101;
		#10;
		en = 1;
		hour_in = 8'b0010_0011;
		#10;
		hour_in = 8'b0001_0010;
		#10;
		hour_in = 8'b0000_0011;
		#10;
		hour_in = 8'b0001_0110;
		#10;
		hour_in = 8'b0000_0101;
		#10;
		hour_in = 8'b0000_0000;
		#10;
		en = 0;
		#10;
		en = 1;

	end
      
endmodule
```


管脚约束如下


```verilog
NET "clk" LOC = B8;
NET "clk" IOSTANDARD = LVCMOS18;
NET "seg[0]" IOSTANDARD = LVCMOS18;
NET "seg[1]" IOSTANDARD = LVCMOS18;
NET "seg[2]" IOSTANDARD = LVCMOS18;
NET "seg[3]" IOSTANDARD = LVCMOS18;
NET "seg[4]" IOSTANDARD = LVCMOS18;
NET "seg[5]" IOSTANDARD = LVCMOS18;
NET "seg[6]" IOSTANDARD = LVCMOS18;
NET "seg[5]" LOC = L13;
NET "seg[4]" LOC = P12;
NET "seg[3]" LOC = N11;
NET "seg[2]" LOC = N14;
NET "seg[1]" LOC = H12;
NET "seg[0]" LOC = L14;
NET "seg[6]" LOC = M12;
NET "sc[3]" LOC = F12;
NET "sc[2]" LOC = J12;
NET "sc[1]" LOC = M13;
NET "sc[0]" LOC = K14;
NET "en" LOC = G12;
NET "led_s" LOC = M5;
NET "sc[0]" IOSTANDARD = LVCMOS18;
NET "sc[1]" IOSTANDARD = LVCMOS18;
NET "sc[2]" IOSTANDARD = LVCMOS18;
NET "sc[3]" IOSTANDARD = LVCMOS18;
NET "en" IOSTANDARD = LVCMOS18;
NET "led_s" IOSTANDARD = LVCMOS18;
# PlanAhead Generated IO constraints 

NET "adj" IOSTANDARD = LVCMOS18;

# PlanAhead Generated physical constraints 

NET "adj" LOC = A7;
NET "alarm" LOC = M4;

# PlanAhead Generated IO constraints 

NET "alarm" IOSTANDARD = LVCMOS18;

# PlanAhead Generated physical constraints 

NET "sw" LOC = C11;

# PlanAhead Generated IO constraints 

NET "sw" IOSTANDARD = LVCMOS18;

# PlanAhead Generated physical constraints 

NET "led_am" LOC = M11;

# PlanAhead Generated IO constraints 

NET "led_am" IOSTANDARD = LVCMOS18;

# PlanAhead Generated physical constraints 

NET "led_chime" LOC = P7;

# PlanAhead Generated IO constraints 

NET "led_chime" IOSTANDARD = LVCMOS18;

# PlanAhead Generated physical constraints 

NET "led_alarm" LOC = P6;

# PlanAhead Generated IO constraints 

NET "led_alarm" IOSTANDARD = LVCMOS18;
# PlanAhead Generated physical constraints 

//NET "led_alarm_sw" LOC = N5;

# PlanAhead Generated IO constraints 

//NET "led_alarm_sw" IOSTANDARD = LVCMOS18;
```

  最终仿真如图 
  24小时

![](http://file.mgek.cc/images/blog/dg-clock-1.webp)
![](http://file.mgek.cc/images/blog/dg-clock-2.webp)
![](http://file.mgek.cc/images/blog/dg-clock-3.webp)
秒进位 分进位
![](http://file.mgek.cc/images/blog/dg-clock-4.webp)
![](http://file.mgek.cc/images/blog/dg-clock-5.webp)
![](http://file.mgek.cc/images/blog/dg-clock-6.webp)
![](http://file.mgek.cc/images/blog/dg-clock-7.webp)
小时进制转换
![](http://file.mgek.cc/images/blog/dg-clock-8.webp)
![](http://file.mgek.cc/images/blog/dg-clock-9.webp)
动态扫描
![](http://file.mgek.cc/images/blog/dg-clock-10.webp)
校时
![](http://file.mgek.cc/images/blog/dg-clock-11.webp) 
数字钟选做要求实现闹钟，正点报时 代码如下


    module alarm(
    input clk,
    input rst,
    input [2:0] ct,
    input [7:0] Hour,
    input [7:0] Minute,
    output [7:0] hour_al,
    output [7:0] minute_al,
    output led_alarm,
    output alarm_state//闹钟开关的状态需传递至display函数在数码管上用0或1显示
    );
    
    reg alarm_sw_cnt=0;//闹钟开关标志，初始时为0
    assign alarm_state =alarm_sw_cnt;
    assign en_hou = ct == 3'b101;//设置闹钟小时状态下en键被7Hz时钟信号扫描到按下时，小时进位，进位信号传递至24进制计数器
    assign en_min = ct == 3'b110;//设置闹钟分钟状态下en键被7Hz时钟信号扫描到按下时，分钟进位，进位信号传递至60进制计数器
    assign en_sw = ct == 3'b111;//设置闹钟开关状态下en键被7Hz时钟信号扫描到按下时，控制闹钟开关状态改变的en_sw被使能
    
    assign led_alarm = (Hour==hour_al)&(Minute==minute_al)&(alarm_sw_cnt);//比较当前时分与设置的闹钟时分，当相等且在闹钟开关标志为1时，闹钟灯长亮1min
    
    always@(posedge clk) begin
    if(en_sw) alarm_sw_cnt = ~alarm_sw_cnt;//改变闹钟开关状态
    end
    
    counter_60 min(clk,rst,en_min,minute_al[7:4],minute_al[3:0],);
    counter_24 hou(clk,rst,en_hou,hour_al[7:4],hour_al[3:0],);
    //设置闹钟状态下计数器的向后进位输出不使用
    
    endmodule


    module chime(
        input clk,
    	 input en_2Hz,
        input [7:0] Hour,
        input [7:0] Minute,
        input [7:0] Second,
        output led_chime
        );
    
    reg [6:0] cnt=0;//7位宽报时计数器，可循环存储0~127
    reg [4:0] num=0;//5位用于存储led_chime的闪烁次数（1~24）
    wire [5:0] dou_num;//将闪烁次数倍增
    
    assign dou_num = num + num;
    assign led_chime = (cnt < dou_num) & cnt[0];//工作时该灯的状态每秒改变两次，由于cnt每秒执行两次+1操作，故设置cnt的个位为1时灯亮，形成闪烁；且cnt>=dou_num后不闪烁
    
    always@(Hour)//将Hour的值转化为闪烁次数
    begin
    	if(Hour==0) num = 5'b11000;//0时报时灯闪烁24次
    	else if(Hour[7:4]==0) num = Hour[3:0];
    	else if(Hour[7:4]==1) num = Hour[3:0] + 5'b01010;
    	else num = Hour[3:0] + 5'b10100;//其余时间用小时数作为闪烁次数
    end
    
    always@(posedge clk)
    begin
    	if(en_2Hz) begin//每秒扫描2次，
    	if(Minute==0) begin//1min内可扫描120次
    		if(Second==0) cnt <= 0;//每个整点时刻先将cnt置零，故报时会在整点过后1s时间开始
    		else cnt <= cnt + 1;//cnt最大为119，之后Minute==0的条件将不满足
    	end
    	end
    end
    
    endmodule

关于FPGA的最终实现： 因为Basys2只有四个数码管，所以秒钟的两位代码里采用的是按钮选择，当按下使能按键时显示出秒钟，使能无效时显示为时钟和分钟 最终fpga实现例图 

![](http://file.mgek.cc/images/blog/dg-clock-12.webp)

数字钟ISE平台代码打包 [https://drive.google.com/open?id=1g2RynQw31bOfzxsSwsKrL5GOCE3AaYrD](https://drive.google.com/open?id=1g2RynQw31bOfzxsSwsKrL5GOCE3AaYrD)
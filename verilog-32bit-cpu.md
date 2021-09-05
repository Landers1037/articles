---
title: verilog编写单周期32位cpu（实现9种逻辑运算）
name: verilog-32bit-cpu
date: 2018-04-14
id: 0
tags: [verilog,verilog学习]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


#### 利用verilog语言编写一个单周期的32位cpu处理简单的运算

实现add，or ，and，jump等九种运算

### 顶层模块top

```verilog
module top(
    input clkin,
    input reset
    );

//wire for rom
wire[31:0] inst;
//wire for controller
wire regdst,jump,branch,memread,memwrite,memtoreg;
wire[3:0] aluop;
wire alusrc,regwrite;
//wire for aluunit
wire zero;
wire[31:0]alures;
wire[31:0]alu_in2; //alu数据输入之前的多路器输出
//wire for memory
wire[31:0] indata;
//wire for register
wire[31:0] rsdata,rtdata;
wire[4:0]rdaddr;
wire[31:0] wrdata;
//wire for pc
wire[31:0] paddr;
//wire for ext
wire[31:0]sign;

irom imem(
 .a(paddr[8:2]),
 .clk(clkin), //上升沿读指令
 .spo(inst));
 
pc pcnt(
  .clkin(clkin),//下降沿改变pc
  .reset(reset),
  .instr(inst),
  .jump(jump),
  .branch(branch),
  .zero(zero),
  .paddr(paddr));
  
ctr mainctr(
  .instr(inst),
  .aluop(aluop),
  .regdst(regdst),
  .alusrc(alusrc),
  .memtoreg(memtoreg),
  .regwrite(regwrite),
  .memread(memread0,
  .memwrite(memwrite),
  .branch(branch),
  .jmp(jump));
 
 alu alu(
   .input1(rsdata),
	.input2(alu_in2),
	.aluctr(aluop),
	.alures(alures),
	.zero(zero));

//alu input2 数据多路器
mux32 mux32_1(
  .in1(sign),
  .in2(rtdata),
  .out(alu_in2),
  .sel(alusrc));
 
dram drem(
  .a(alures[7:2]),
  .d(rtdata),
  .clk(!clkin),//下降沿写数据
  .we(memwrite),
  .spo(indata));
 
//reg 写数据多路器
mux32 mux32_2(
  .in1(indata),
  .in2(alures),
  .out(wrdata),
  .sel(memtoreg));

regfile regfile(
  .rsaddr(inst[25:21]),
  .rtaddr(inst[20:16]),
  .clk(!clkin),//下降沿写reg
  .reset(reset),
  .regwriteaddr(rdaddr),
  .regwritedata(wrdata),
  .regwriteen(regwrite),
  .rsdata(radata),
  .rtdata(rtdata));
  
//reg 地址选择多路器
mux5 mux5_1(
  .in1(inst[15:11]),
  .in2(inst[20:16]),
  .out(rdaddr),
  .sel(regdst));

//sign extend扩展
signext signext(
  .inst(inst[15:0]),
  .data(sign));
  
 endmodule
```


​    

### 符号扩展

```verilog
module signext(
    input [15:0] inst,
    output [31:0] data
    );//sign ectend
assign data=inst[15:15]?{16'hffff,inst}:{16'h0000,inst};

endmodule
```


### 控制器

```verilog
module control(//control
    input [31:0]instr,
    output regdst,
    output alusrc,
    output memtoreg,
    output regwrite,
    output memread,
    output memwrite,
    output branch,
    output [3:0] aluop,
    output jmp
    );

wire [5:0]opcode;
wire [5:0]funct;
wire [11:0] ctr_varl;

reg[3:0] aluop;
reg regdst;
reg alusrc;
reg memtoreg;
reg regwrite;
reg memread;
reg memwrite;
reg branch;
reg jmp;
assign opcode=instr[31:26];
assign funct=instr[5:0];
assign ctr_varl={opcode,funct};
always@(ctr_varl)
  begin
    casex(ctr_varl)
	  12'b000000xxxxxx:
	   begin
		 case(funct)
	   6'b1000000:  //add
		  aluop=4'b0001;
		6'b100010://sub
		  aluop=4'b0010;
		6'b100100://and
		  aluop=4'b0011;
		6'b100101://or
		  aluop=4'b0100;
		6'b101010://slt
		  aluop=4'b0101;
    
		default:
		 aluop=4'b0000;
		endcase//r ins
		regdst=1;
		alusrc=0;
		memtoreg=0;
		regwrite=1;
		memread=0;
		memwrite=0;
		branch=0;
		jmp=0;
		
	end
  12'b100011xxxxxx://lw
  begin
   aluop=4'b0001;
	regdst=0;
	alusrc=1;
	memtoreg=1;
	regwrite=1;
	memread=1;
	memwrite=0;
	branch=0;
	jmp=0;
 end
 
  12'b101011xxxxxx://sw
  begin
    aluop=4'b0001;
	 regdst=0;
	 alusrc=1;
	 memtoreg=0;
	 regwrite=0;
	 memread=0;
	 memwrite=1;
	 branch=0;
	 jmp=0;
	end
 
   6'b000100://beq
  begin
   aluop=4'b0010;
	regdst=0;
	jmp=0;
	branch=1;
	memtoreg=0;
	alusrc=0;
	regwrite=0;
	memwrite=0;
	memread=0;
   end
 6'b000010://j
  begin 
   aluop=4'b0101;
	regdst=0;
	jmp=1;
	branch=0;
	memtoreg=0;
	alusrc=0;
	regwrite=0;
	memwrite=0;
	memread=0;
	end
	default:
	begin
	 aluop=4'b0000;
	 regdst=0;
	 alusrc=0;
	 memtoreg=0;
	 regwrite=0;
	 memread=0;
	 memwrite=0;
	 branch=0;
	 jmp=0;
	 end
endcase
end
```


​     
    
    
```verilog
endmodule
```


### 运算器alu

    module alu(
        input [31:0] input1,
        input [31:0] input2,
        input [3:0] aluctr,
        input [31:0] alures,
        output zero
        );
    
    reg zero;
    reg[31:0]alures;
    always@(input1 or input2 or aluctr)
       begin
    	  case(aluctr)
    	     4'b0001:
    		   alures=input1+input2;
    		  4'b0010:
    		   begin
    			  alures=input1-inpuit2;
    			  if(alures==0)
    			    zero=1;
    			  else
    			   zero=0;
    			end
    		  4'b0011:
    		    alures=input1&input2;
    		  4'b0100:
    		    alures=input1|input2;
    		  4'b0101:
    		    begin
    			  if(input1<input2)
    			    alures=1;
    			 else 
    			    alures=0;
    				end
    			default:
    			  alures=0;
    			 endcase
    			end
    		endmodule


​    
    

### 选择器32

    module mux32(
        input [31:0] in1,
        input [31:0] in2,
        input sel,
        input [31:0] out
        );
    //sel=1:out=in1
    //sel=0:out=in2
    assign out=sel?in1;in2;
    
    endmodule


### 选择器5

    module mux5(
        input [4:0] in1,
        input [4:0] in2,
        input sel,
        input [4:0] out
        );
    //sel=1:out=in1
    //sel=0:out=in2
    assign out=sel?in1:in2;
    
    endmodule


### 寄存器

```verilog
module regfile(
    input clk,
    input reset,
    input [31:0] regwritedata,
    input [4:0] regwriteaddr,
    input regwriteen,
    output [31:0] rsdata,
    output [31:0] rtdata,
    input [4:0] rsaddr,
    input [4:0] rtaddr
    );

reg[31:0] regs[31:0];
assign rsdata=(rsaddr==5'b0)?32'b0:regs[rsaddr];
assign rtdata=(rtaddr==5'b0)?32'b0:regs[rtaddr];
integer i_reg;
always@(posedge clk)
begin
  if(!reset)
     begin 
	    for (i_reg=0;i_reg<32;i_reg=i_reg+1)
		   begin
			  regs[i_reg]=i_reg;
			end
		i_reg=0;
	end
 else
   begin
	if(regwriteen==1)
	 begin
	  regs[regwriteaddr]=regwritedata;
	  end
	 end
end
endmodule
```


经过测试，部分代码有错，可以参考cpu输出学号的代码 [https://www.liaorenjie.top/Landers/2018/628.html](https://www.liaorenjie.top/Landers/2018/628.html)
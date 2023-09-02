---
title: verilog语言将学号输入到ip内核中
name: verilog-homework1
date: 2018-05-11
id: 0
tags: [mips汇编学习,verilog,verilog学习]
categories: []
abstract: ""
---


#### 使用mips语言将底层的汇编指令输入到cpu的内部

mips代码示例（使用qtspim仿真）

> main: addi $2,$0,55 sw $2,0($3) addi $2,$0,48 sw $2,4($3) addi $2,$0,48 sw $2,8($3) addi $2,$0,48 sw $2,12($3) addi $2,$0,48 sw $2,16($3) addi $2,$0,48 sw $2,20($3) addi $2,$0,48 sw $2,24($3) addi $2,$0,48 sw $2,28($3) addi $2,$0,48 sw $2,32($3) addi $2,$0,48 sw $2,36($3) j main 利用addi指令和sw指令将学号的ascall码写入到存储中

放入到qtspim中生成机器码指令集

> MEMORY\_INITIALIZATION\_RADIX=16; MEMORY\_INITIALIZATION\_VECTOR= 20020055, ac620000, 20020030, ac620004 , 20020030, ac620008, 20020030, ac62000c, 20020030, ac620010, 20020030, ac620014, 20020030, ac620018, 20020030, ac62001c, 20020030, ac620020, 20020030, ac620024, 08100000,

#### 新建verilog项目，建立ip内核为rom和ram

选择128位 32位rom内核 从生成的coe文件中导入指令集 ![C0xWxx.md.png](https://s1.ax1x.com/2018/05/11/C0xWxx.md.png) ![C0xRR1.md.png](https://s1.ax1x.com/2018/05/11/C0xRR1.md.png)

> **代码示例**

```verilog
module top(
    input clkin,
    input reset
    );
	 
	 reg [31:0] pc,add4;
	 wire choose4;//4位选择器
	 wire [31:0] expand2,mux2,mux3,mux4,mux5,address,jmpaddr,inst;
	 wire [4:0]mux1;//选择器
	 //write for controller
	 wire reg_dst,jmp,branch,memread,memwrite,memtoreg;
	 wire [1:0] aluop;
	 wire alu_src,regwrite;
	 //write for alunit
	 wire zero;
	 wire [31:0] aluRes;
	 //write for aluctr
	 wire [3:0]aluCtr;
	 //write for memory
	 wire [31:0]memreaddata;//memory data
	 //write for register
	 wire [31:0]RsData,RtData;//regfile data
	 //wireforext
	 wire [31:0]expand;
	 
	 always @(negedge clkin)
	 begin
		if(~reset)	begin
			pc=mux5;
			add4=pc+4;
			end
		else
			begin
				pc=32'b0;
				add4=32'h4;
			end
		end
		
		ctr mainctr(
			.opcode(inst[31:26]),
			.regDst(reg_dst),
			.aluSrc(alu_src),
			.memToReg(memtoreg),
			.regWrite(regwrite),
			.memRead(memread),
			.memWrite(memwrite),
			.branch(branch),
			.aluop(aluop),
			.jmp(jmp));
			
		alu alu(.input1(RsData),
				.input2(mux2),
				.aluCtr(aluCtr),
				.zero(zero),
				.aluRes(aluRes));
				
		aluctr aluctr1(.ALUOp(aluop),
					.funct(inst[5:0]),
					.ALUCtr(aluCtr));
					
		dram dmem(
				.a(aluRes[7:2]),
				.d(RtData),
				.clk(!clkin),
				.we(memwrite),
				.spo(memreaddata)
				);
				
		irom imem(
				.a(pc[8:2]),
				.clk(clkin),
				.spo(inst));
				
		regFile regfile(
				.RsAddr(inst[25:21]),
				.RtAddr(inst[20:16]),
				.clk(!clkin),
				.reset(reset),
				.regWriteAddr(mux1),
				.regWriteData(mux3),
				.regWriteEn(regwrite),
				.RsData(RsData),
				.RtData(RtData)
				);
				
		signext signext(.inst(inst[15:0]),.data(expand));
		
		assign mux1=reg_dst?inst[15:11]:inst[20:16];
		assign mux2=alu_src?expand:RtData;
		assign mux3=memtoreg?memreaddata:aluRes;
		assign mux4=choose4?address:add4;
		assign mux5=jmp?jmpaddr:mux4;
		assign choose4=branch&zero;
		assign expand2=expand<<2;
		assign jmpaddr={add4[31:28],inst[25:0],2'b00};
		assign address=pc+expand2;
```


​    
```verilog
endmodule

module ctr(
    input [5:0] opcode,
    output regDst,
    output aluSrc,
    output memToReg,
    output regWrite,
    output memRead,
    output memWrite,
    output branch,
    output [1:0] aluop,
    output jmp
    );
	 reg regDst;
	 reg aluSrc;
	 reg memToReg;
	 reg regWrite;
	 reg memRead;
	 reg memWrite;
	 reg branch;
	 reg [1:0] aluop;
	 reg jmp;
	 
always @(opcode) begin

	case(opcode)
		//jmp 
		6'b000010: 
		begin
			regDst=0;
			aluSrc=0;
			memToReg=0;
			regWrite=0;
			memRead=0;
			memWrite=0;
			branch=0;
			aluop=2'b00;
			jmp=1;
		end
		//add,sub,and,or,slt
		6'b000000: 
		begin
			regDst=1;
			aluSrc=0;
			memToReg=0;
			regWrite=1;
			memRead=0;
			memWrite=0;
			branch=0;
			aluop=2'b10;
			jmp=0;
		end
		//lw
		6'b100011:
		begin
			regDst=0;
			aluSrc=1;
			memToReg=1;
			regWrite=1;
			memRead=1;
			memWrite=0;
			branch=0;
			aluop=2'b00;
			jmp=0;
		end
		//sw
		6'b101011:
		begin
			regDst=0;		//donnot care
			memToReg=0;		//donnot care
			aluSrc=1;
			regWrite=0;
			memRead=0;
			memWrite=1;
			branch=0;
			aluop=2'b00;
			jmp=0;
		end
		//beq
		6'b000100:
		begin
			regDst=0;		//donnot care
			memToReg=0;		//donnot care
			aluSrc=0;
			regWrite=0;
			memRead=0;
			memWrite=0;
			branch=1;
			aluop=2'b01;
			jmp=0;
		end
		//addi
		6'b001000:
		begin
			regDst=0;		//donnot care
			memRead=0;		//donnot care
			memWrite=0;		//donnot care
			aluSrc=1;
			memToReg=0;
			regWrite=1;
			branch=0;
			aluop=2'b00;		
			jmp=0;
		end
```


​    		
​    		
​    		
​    
​    
```verilog
	endcase
	
	end

endmodule

module alu(
    input [31:0] input1,
    input [31:0] input2,
    input [3:0] aluCtr,
    output [31:0] aluRes,
    output zero
    );
	 reg zero;
	 reg [31:0] aluRes;
	 
	 always @(input1,input2,aluRes)
		begin
		zero=0;
		case(aluCtr)
		4'b0110:
			begin
				aluRes=input1-input2;
				if(aluRes==0)	zero=1;	else	zero=0;
			end
		4'b0010: aluRes=input1+input2;
		4'b0000:	aluRes=input1&input2;
		4'b0001:	aluRes=input1|input2;
		4'b1100:	aluRes=~(input1|input2);
		4'b0111:
		begin
			if(input1<input2)	aluRes=1;
		end
		default:
			aluRes=0;
		endcase
		
end

endmodule

module aluctr(
    input [1:0] ALUOp,
    input [5:0] funct,
    output [3:0] ALUCtr
    );
	 reg [3:0] ALUCtr;
	 always @(ALUOp,funct)
	 casex({ALUOp,funct})
		8'b00xxxxxx:ALUCtr=4'b0010;
		8'b01xxxxxx:ALUCtr=4'b0110;
		8'b1xxx0000:ALUCtr=4'b0010;
		8'b1xxx0010:ALUCtr=4'b0110;
		8'b1xxx0100:ALUCtr=4'b0000;
		8'b1xxx0101:ALUCtr=4'b0001;
		8'b1xxx1010:ALUCtr=4'b0111;
		endcase
```


​    	
​    
​    
```verilog
endmodule

module regFile(
    input clk,
    input reset,
    input [31:0] regWriteData,
    input [4:0] regWriteAddr,
    input regWriteEn,
    output [31:0] RsData,
    output [31:0] RtData,
    input [4:0] RsAddr,
    input [4:0] RtAddr
    );
reg [31:0] regs[0:31];

assign RsData = (RsAddr==5'b0)?32'b0:regs[RsAddr];
assign RtData = (RtAddr==5'b0)?32'b0:regs[RtAddr];

integer i;

always @(posedge clk) begin
	if(!reset) begin
		if(regWriteEn==1) 
			regs[regWriteAddr]=regWriteData;
	end
	
	else	begin
		for(i=0;i<32;i=i+1)
			regs[i]=0;
	end
end
			
endmodule

module signext(
    input [15:0] inst,
    output [31:0] data
    );
assign data=inst[15:15]?{16'hffff,inst}:{16'h0000,inst};

endmodule
regfiled的test代码
module regsim;

            // Inputs
            reg clk;
            reg reset;
            reg [31:0] regWriteData;
            reg [4:0] regWriteAddr;
            reg regWriteEn;
            reg [4:0] RsAddr;
            reg [4:0] RtAddr;
```


​    
```verilog
 // Outputs

           wire [31:0] RsData;
           wire [31:0] RtData;

 // Instantiate the Unit Under Test (UUT)

            regFile uut (

                       .clk(clk),

                       .reset(reset),

                       .regWriteData(regWriteData),

                       .regWriteAddr(regWriteAddr),

                       .regWriteEn(regWriteEn),

                       .RsData(RsData),

                       .RtData(RtData),

                       .RsAddr(RsAddr),

                       .RtAddr(RtAddr)

            );

parameter PERIOD=20;

                       always begin

                                   clk=0;

                                   #(PERIOD/2) clk=1;

                                   #(PERIOD/2);

                       end

integer i;

            initial begin

                                   // Initialize Inputs

                       clk = 0;

                       reset = 0;

                       regWriteData = 0;

                       regWriteAddr = 0;

                       regWriteEn = 0;

                       RsAddr = 0;

                       RtAddr = 0;
```


​    
​    
```verilog
                       // Wait 100 ns for global reset to finish

                       #100;

       // Add stimulus here

                        regWriteData=32'h55aaaa55;

                        regWriteEn=1;

                        reset=1;

                        #100;

                        reset=0;

                        end
```


​                          
​    
```verilog
                        always begin

                                   for(i=31;i>=1;i=i-1)begin

                                   regWriteAddr=i;

                                   RsAddr=i;

                                   #PERIOD;

                                   end

                       end
```


​                          
​    
```verilog
endmodule
```






#### 最后仿真topsim，观察rom中的学号 按照步骤，选择top、utt文件中的imem观察spo信号

#### 将spo转换为ascll值

![](http://file.mgek.cc/images/blog/verilog-homework1-1.webp)
**将时间域设置为1s使学号完全输出** 
![](http://file.mgek.cc/images/blog/verilog-homework1-2.webp)
**最终的结果**
![](http://file.mgek.cc/images/blog/verilog-homework1-3.webp)
**对regfile模块的仿真** 
![](http://file.mgek.cc/images/blog/verilog-homework1-4.webp)
![](http://file.mgek.cc/images/blog/verilog-homework1-5.webp)

#### 实验结果分析

*   spo是什么：spo为32位数值实际为输入的指令，在定义的mips指令里我们只添加了addi和sw指令
*   为什么学号的每位间有额外数字b，因为当we有效时为写入信号，将学号的每一位输入到存储里。当we无效时从寄存器2里读取数据，执行了addi指令，addi指令的32位中对应数字的那部分转化为ascll就是b
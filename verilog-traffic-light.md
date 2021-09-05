---
title: verilog实现交通灯
name: verilog-traffic-light
date: 2018-03-27
id: 0
tags: [verilog,技术]
categories: []
abstract: ""
---
<code>Sorry</code>该文章暂无概述💊
<!--more-->


感谢某吴姓大佬

    module transport_light(
        input CP,
    	 input CR,
        output reg  [1:0]A,
        output reg  [1:0]B
        );
    	 
    	 reg [1:0] S;
    	 
    	 always @ (posedge CP)
    	    begin
    		    if(!CR)
    			 begin
    		    S <= 2'b00;
    			 A <= 2'b00;
    			 B <= 2'b00;
    			 end


​    			
    			 else
    			   case (S)
    				   2'b00: begin A <=2'b00 ;B <= 2'b10 ;S <= S+1;end
    					2'b01: begin A <=2'b11 ;B <= 2'b11 ;S <= S+1;end
    					2'b10: begin A <=2'b10 ;B <= 2'b00 ;S <= S+1;end
    					2'b11: begin A <=2'b11 ;B <= 2'b11 ;S <= 2'b00;end
    				endcase
    		 end		


​    
​    
```verilog
endmodule

TEST BENCH

module test;

	// Inputs
	reg CP;
	reg CR;

	// Outputs
	wire [1:0] A;
	wire [1:0] B;

	// Instantiate the Unit Under Test (UUT)
	transport_light uut (
		.CP(CP), 
		.CR(CR), 
		.A(A), 
		.B(B)
	);
	
	always begin
	
	   CP = 1;
		#150;
		CP = 0;
		#150;
		CP = 1;
		#25;
		CP = 0;
		#25;
		CP = 1;
		#100;
		CP = 0;
		#100;
		CP = 1;
		#25;
		CP = 0;
		#25;
		
	end

	initial begin
		// Initialize Inputs
		CP = 0;
		CR = 0;

		// Wait 100 ns for global reset to finish
		#100;
		CR = 1;
        
		// Add stimulus here

	end
      
endmodule
```
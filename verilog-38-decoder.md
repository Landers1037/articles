---
title: verilog实现3-8译码
name: verilog-38-decoder
date: 2018-04-15
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


```verilog
module yima38(A,en,Q
    );
input [2:0]A;
input en;
output reg[7:0] Q;
integer k;
always@(A,en)begin
Q=8'b1111_1111;
for(k=0;k<=7;k=k+1)
   if((en==1)&&A==k)
	   Q[k]=0;
	else
      Q[k]=1;
end		
```


​    
<!--more-->


```verilog
module yima38(A,en,Q
    );
input [2:0]A;
input en;
output reg[7:0] Q;
integer k;
always@(A,en)begin
Q=8'b1111_1111;
for(k=0;k<=7;k=k+1)
   if((en==1)&&A==k)
	   Q[k]=0;
	else
      Q[k]=1;
end		
```


​    <!--more-->
```verilog
endmodule
```


3-8线译码器打包 [https://drive.google.com/open?id=1MzxKVJcPlgS\_2bPA37XZuT\_gkzn3nrbZ](https://drive.google.com/open?id=1MzxKVJcPlgS_2bPA37XZuT_gkzn3nrbZ)
---
title: verilog实现7人投票器
name: verilog-voter
date: 2018-03-27
id: 0
tags: [verilog]
categories: []
abstract: ""
---


```verilog
module people7(s,sum,vote
    );
input [6:0]s;
output reg vote;//定义1为通过，0为禁止
output reg[2:0] sum;//对7人结果的统计票数大于等于4为通过
integer k;
always@(s) begin
sum=0;

for (k=0;k<=6;k=k+1)
   begin
	if(s[k]) sum=sum+1;
	  if(sum>3) vote<=1;
	  else vote<=0;
   end

 end
endmodule   
```

<!--more-->

7人投票器打包 [https://drive.google.com/open?id=1kP50XV2VeEsLHB9JCXDWeDa8PAxXGW7V](https://drive.google.com/open?id=1kP50XV2VeEsLHB9JCXDWeDa8PAxXGW7V)
---
title: verilog语法学习1
name: verilog-learn1
date: 2018-03-27
id: 0
tags: [verilog,verilog学习]
categories: []
abstract: ""
---


使用//进行简单行代码注释 使用/*和*/将一段代码注释 例如<!--more-->

```verilog
/*module SEG7_LUT_8(HEX0,HEX1,HEX2,HEX3,HEX4,HEX5,HEX6,HEX7,iDIG);
  input[31:0] iDIG;
  output[6:0] HEX0,HEX1,HEX2,HEX3,HEX4,HEX5,HEX6,HEX7;
  SEG7_LUTu0(HEX0,iDIG[3:0]);
  SEG7_LUTu1(HEX1,iDIG[7:4]);
  SEG7_LUTu2(HEX2,iDIG[11:8]);
  SEG7_LUTu3(HEX3,iDIG[15:12]);
  SEG7_LUTu4(HEX4,iDIG[19:16]);
  SEG7_LUTu5(HEX5,iDIG[23:20]);
  SEG7_LUTu6(HEX6,iDIG[27:24]);
  SEG7_LUTu7(HEX7,iDIG[31:28]);
endmodule*/
```

  即整段代码被注释
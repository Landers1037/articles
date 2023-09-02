---
title: matlab信号处理-BPSK调制
name: matlab4
date: 2019-05-14
id: 0
tags: [matlab]
categories: []
abstract: ""
---


```matlab
clear
close all

%%%%% 参数定义
%  系统参数定义
sys_param = SystemParaDef();
%  输入的数据流
input_data = [0 1 0 0 1 0 1 1 0 0 1 1 1 0 0 1 0 1 0 1];
```

<!--more-->

```matlab
%%%% 仿真计算
% ----------信号发射端部分
% 信道编码
% *******
% 信号调制
modulated_signal = BPSKMudulation(input_data,sys_param);
% 带通滤波
% *******
% 封装成帧结构
% *******


% 画发射射频信号的频域图
figure()
PlotFFTSignal(modulated_signal,sys_param.sim_freq);
title('发送信号频谱图');


% ----------信道部分
% 添加信道噪声
% *******

% ----------信号接收端部分
% 采用正交解调方式
% 获得Q路信号的基带信号
baseband_signal = QPhaseDemodulation(modulated_signal,sys_param);
% 对基带信号采样
sampled_signal = ReceiverSampling(baseband_signal,sys_param);
% 画采样信号的时域图
figure()
PlotTDSignal(sampled_signal,sys_param.sample_freq,sys_param.bit_rate);
title('接收信号的时域图');
% 解码
decode_data = BPSKDecoder(sampled_signal,sys_param);

% ----------统计性能
error_ratio = CalBitErrorRate(input_data,decode_data)
```


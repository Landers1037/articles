---
title: matlab的QPSK信号调制与解调
name: matlab5
date: 2019-05-17
id: 0
tags: [matlab]
categories: []
abstract: ""
---


## main文件部分

源代码地址 [Gitee](https://gitee.com/lrjgood/qpsk_modulation_of_matlab)


<!--more-->
## main文件部分

源代码地址 [Gitee](https://gitee.com/lrjgood/qpsk_modulation_of_matlab)


<!--more-->


## main文件部分

源代码地址 [Gitee](https://gitee.com/lrjgood/qpsk_modulation_of_matlab)

<!--more-->

```matlab
clear
close all

%%%%% 参数定义
%  系统参数定义
sys_param = SystemParaDef();
%  输入的数据流
input_data = [0 1 0 0 1 0 1 1 0 0 1 1 1 0 0 1 0 1 0 1];

%%%% 仿真计算
% ----------信号发射端部分
% 信道编码
% *******
% 信号调制
modulated_signal = QPSKMudulation(input_data,sys_param);
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
modulated_signal = modulated_signal + 0.2*rand(1,192000);
% ----------信号接收端部分
% 采用正交解调方式
% 获得Q路信号的基带信号
baseband_signal=QPhaseDemodulation(modulated_signal,sys_param,0);
baseband_signa2=QPhaseDemodulation(modulated_signal,sys_param,pi/2);
%result_signal = [baseband_signal,baseband_signa2];

% 对基带信号采样
sampled_signal = ReceiverSampling(baseband_signal,sys_param);
sampled_signal2 = ReceiverSampling(baseband_signa2,sys_param);

figure()
plot(baseband_signal,baseband_signa2,'go');
title('星座图');

%并串转换
zeros(1,[]);
for k=0:8
    result(1+k*200:100+k*200) = sampled_signal(1+100*k:100+100*k);
    result(101+k*200:200+k*200) = sampled_signal2(1+100*k:100+100*k);
end
result(1801:1899) = sampled_signal(901:999);
result(1900:1998) = sampled_signal2(901:999);
    
% 解码
decode_data1 = BPSKDecoder(sampled_signal,sys_param);
decode_data2 = BPSKDecoder(sampled_signal2,sys_param);
data_result = reshape([decode_data1;decode_data2],1,[]);
% 画采样信号的时域图
figure()
PlotTDSignal(result,sys_param.sample_freq,sys_param.bit_rate);
title('接收信号的时域图');

% ----------统计性能
error_ratio = CalBitErrorRate(input_data,data_result)

```

## 调制部分

```matlab
function modulated_signal = QPSKMudulation(input,sys_param)
%变为qpsk调制 使用四个相位调制
% 获取数据长度
len = length(input);
% 一个比特的长度
symbol_len = sys_param.sim_freq/sys_param.bit_rate;
%reshape信号为并行信号
bingxing = reshape(input,2,[]);
% 初始化
modulated_signal = [];
% 离散时间点
t_i = 1:symbol_len;
% 获得符号表达
for k = 1:len/2
     if bingxing(1,k) == 0 && bingxing(2,k) == 0
        bit_representation = -cos(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i)+sin(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i);
        
    end
    if bingxing(1,k) == 0 && bingxing(2,k) == 1
        bit_representation = -cos(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i)-sin(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i);
    end
    if bingxing(1,k) == 1 && bingxing(2,k) == 0
        bit_representation = cos(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i)+sin(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i);
    end
    if bingxing(1,k) == 1 && bingxing(2,k) == 1
        bit_representation = cos(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i)-sin(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i);
    end
    
    modulated_signal = [modulated_signal bit_representation];
end
```

## 解调部分

```matlab
function output_signal = QPhaseDemodulation(input_signal,sys_param,p)

% 获取输入信号的长度
signal_len = length(input_signal);
t_i = 1:signal_len;
% 产生Q路相关解调信号cos(wt)
Q_t = cos(2*pi*sys_param.center_freq/sys_param.sim_freq*t_i+p);
s_t = input_signal.*Q_t;

output_signal = LowPassFilter(s_t,1/4);
%%%%%%%%
function sampled_signal = ReceiverSampling(orignal_signal,sys_param)

% 采样的比例(每间隔sampling_ratio个点采样一个)
sampling_ratio = sys_param.sim_freq/sys_param.sample_freq;

% 数据的总点数
data_len = length(orignal_signal);
%采样后的点数
sampled_len = floor(data_len/sampling_ratio);
% 间隔采样
sample_location = 1 + 0:sampling_ratio:sampling_ratio*(sampled_len-1);
% 采样
sampled_signal = orignal_signal(sample_location);
```

## 实验结果

![](/images/matlab5.webp)


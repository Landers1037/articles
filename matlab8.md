---
title: matlab仿真对信号的扩频，解扩，解调
name: matlab8
date: 2019-06-07
id: 0
tags: [matlab]
categories: []
abstract: ""
---
使用matlab仿真，对信号进行11倍扩频，使用直接扩频的方法，结果与BPSK调制的结果对比
给输入的信号添加一个窄带白噪声信号，观察误码率

环境：Matlab2016a


<!--more-->


使用matlab仿真，对信号进行11倍扩频，使用直接扩频的方法，结果与BPSK调制的结果对比
给输入的信号添加一个窄带白噪声信号，观察误码率

环境：Matlab2016a

<!--more-->

本实验基于前面的BPSK调制，使用其部分库函数，修改并增加了部分库函数

#### main

```matlab
%bpsk的11倍扩频

clear
close all 
%%%%% 参数定义
%  系统参数定义
sys_param = SystemParaDef();
%  输入的数据流
input_data = [0 1 0 0 1 0 1 1 0 0 1 1 1 0 0 1 0 1 0 1];
x = input_data;

%%信号的扩频部分
L = MM();
M = DMM(L);

%扩频

ss_signal = Bandexpand(input_data,L);

%调制信号
modulated_signal_before = BPSKMudulation(input_data,sys_param,0);
modulated_signal_after = BPSKMudulation(ss_signal,sys_param,1);

% 画发射射频信号的频域图
figure()
subplot(2,1,1);
PlotFFTSignal(modulated_signal_before,sys_param.sim_freq,sys_param.center_freq);
title('扩频前调制信号频谱图');
subplot(2,1,2);
PlotFFTSignal(modulated_signal_after,sys_param.sim_freq,sys_param.center_freq);
title('扩频后调制信号频谱图');

%产生高斯噪声
noise = addn(modulated_signal_after,sys_param);
modulated_signal_no =modulated_signal_after+ 50.*noise;
figure()
PlotFFTSignal(modulated_signal_no,sys_param.sim_freq,sys_param.center_freq);
title('添加窄带噪声信号频谱图');

%滤波
baseband_signal = QPhaseDemodulation(modulated_signal_after,sys_param);
%解扩
ds_signal = Dbandex(baseband_signal,M);
%采样
sampled_signal = ReceiverSampling(ds_signal,sys_param);

%扩频前后的时域图
% figure()
% subplot(2,1,1)
% plot(modulated_signal_before(150000:150100))
% title('扩频前的时域图')
% subplot(2,1,2)
% plot(modulated_signal_after(2000000:2000500))
% title('扩频后的时域图')


% 无噪声信号解码
decode_data = BPSKDecoder(sampled_signal,sys_param);


%滤波
baseband_signal2 = QPhaseDemodulation(modulated_signal_no,sys_param);
%加噪声解扩
ds_signal2 = Dbandex(baseband_signal2,M);
%采样
sampled_signal2 = ReceiverSampling(ds_signal2,sys_param);
% 加噪声信号解码
decode_data_no = BPSKDecoder(sampled_signal2,sys_param);
% figure()
% PlotTDSignal(sampled_signal2,sys_param.sample_freq,sys_param.bit_rate);
% title('加噪声解扩后的采样信号的时域图');

%解码后的信号序列
figure()
t=0:19;
stem(t,decode_data);
title('解码后的二进制信息序列');

error_ratio=CalBitErrorRate(input_data,decode_data_no)
```

##### 函数解释

- MM() 生成用于11倍扩频的M序列
- DMM() 生成用于解扩的序列
- Bandexpand() 扩频函数，传入原始信号序列和M序列
- BPSKMudulation() 调制函数，对输入的信号进行BPSK调制
- Dbandex() 解扩函数

#### M序列的生成

```matlab
function L = MM()

x1=0;x2=0;x3=1;
m=220; %20*11
for i=1:m
    y3=x3; y2=x2; y1=x1;
    x3=y2; x2=y1;
    x1=xor(y3,y1);
    L(i)=y1;
end
```

#### DM序列的生成

```matlab
function M = DMM(L)
%%解扩序列
m = 220;
for i=1:m
    M(i)=1-2*L(i); %转化为双极性
end
K = 1:1:m;
figure(1)
subplot(2,1,1);
stem(K-1,M);
axis([0,11,-1,1])
xlabel('k');
ylabel('M解扩序列');
```

#### 扩频

```matlab
function s = Bandexpand(input_data,L)
x = input_data;
figure()
subplot(2,1,1);
t=0:19;
stem(t,input_data);
title('扩频序列的生成');
tt=0:219;
subplot(2,1,2)
l=1:11*20;
y(l)=1;
for i=1:20
    k=11*i-10;
    for j=1:11   
        y(k)=x(i);
        k=k+1;
    end
    y(k)=x(i);
end

s(l)=0;
for i=1:220
    s(i)=xor(L(i),y(i));
end  
stem(tt,s);
axis([0,220,0,1]);
```

#### 解扩

```matlab
function ds_signal = Dbandex(baseband_signal,M)

len = length(baseband_signal);
count = len/220;
MM = rectpulse(M,count);
ds_signal = MM(1:len).*baseband_signal();
```

#### 窄带噪声信号的产生

```matlab
function noise = addn(modulated_signal_after,sys_param)

noise=rand(1,length(modulated_signal_after));
BPF=fir1(48,[2*2.2e9/sys_param.sim_freq 2*2.4e9/sys_param.sim_freq]);
noise=filter(BPF,1,noise);
```

#### 系统参数

```matlab
function sys_param = SystemParaDef()

%%系统类参数定义sys_param

% 射频信号的中心频率
sys_param.center_freq = 2.4e9;

% 仿真信号采样频率
sys_param.sim_freq = sys_param.center_freq*8;


% 基带信号采样频率
sys_param.sample_freq = 1e8;
% 信号发送带宽
sys_param.band = 2*10^6;
% 信号比特速率
sys_param.bit_rate = 1*10^6;
% 仿真的信噪比(dB)
sys_param.SNR = 20;

% 一个比特的时间长度
sys_param.bit_duration = 1/sys_param.bit_rate;
% 采样时间间隔
sys_param.sampling_interval = 1/sys_param.sample_freq;
```

#### 仿真结果

![](/images/matlab8-1.webp)

![](/images/matlab8-2.webp)

![](/images/matlab8-3.webp)
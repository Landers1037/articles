---
title: matlab仿真矩量法计算板间电容
name: matlab6
date: 2019-05-18
id: 0
tags: [matlab]
categories: []
abstract: ""
---


## 题目

![](/images/matlab6-1.webp)


<!--more-->
## 题目

![](/images/matlab6-1.webp)


<!--more-->


## 题目

![](/images/matlab6-1.webp)

<!--more-->
![](/images/matlab6-2.webp)

## 计算

```matlab
NN = [3 7 11 18 39 59];
disp(' ')
disp('迭代次数      输出')
for N = NN
	CL=3.0e8; %光速
	ER=1.0;
	EO=8.8541878176E-12;%真空介电常数
	H=2.0;
	W=5.0;
	
	NT=2*N;
	DELTA=W/(N);%单位长度
	
	for K=1:N
	X(K) = DELTA*(K - .5);
	Y(K) = -H/2.0;%分上下表面
	X(K+N) = X(K);
	Y(K+N) = H/2.0;
	end
	
	FACTOR = DELTA/(2.0*pi*EO);%前面的系数
	
	for I=1:NT
		for J=1:NT
			if(I==J)
				A(I,J) = -(log(DELTA) - 1.5)*FACTOR;
			else
				R = sqrt((X(I) - X(J))^2 + (Y(I)-Y(J))^2);
				A(I,J) = -log(R)*FACTOR;
			end
		end
	end
	
	for K=1:N
		B(K)=1.0;%上表面的电势
		B(K+N) = -1.0;%下表面的
	end
	
	
	NIV = NT;
	NMAX=100;
	A = inv(A);
	for I=1:NT
		RO(I)=0.0;
		for M=1:NT
			RO(I)=RO(I)+A(I,M)*B(M); %p的矩阵
		end
	end
	
	SUM=0.0;
	for I=1:N
		SUM = SUM+RO(I);%求上表面的电荷密度
	end
	Q=SUM*DELTA;%求出单位密度的电荷量
	VD=2.0;
	C=abs(Q)/VD;%求出单位长度的电容大小
	ZO = sqrt(ER)/(CL*C);%根据电容求出微带线的特性阻抗
	
	disp(num2str([N ZO ]))
	
	end
```


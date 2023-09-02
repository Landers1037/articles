---
title: arm-linux-gcc交叉编译器的安装
name: linux-arm-gcc
date: 2019-04-26
id: 0
tags: [linux]
categories: []
abstract: ""
---


# 环境ubuntu16.04

## 下载gcc编译器

[官网地址](https://releases.linaro.org/components/toolchain/binaries/6.1-2016.08/arm-linux-gnueabihf/gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf.tar.xz)

## 放到ubuntu的下载目录下

```bash
#右键 提取到此处
得到如下文件夹 gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf
```

<!--more-->

## 新建一个文件夹放置gcc

```bash
sudo mkdir /usr/local/arm-gcc
#在下载目录下打开终端
sudo mv gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf /usr/local/arm-gcc
#把gcc文件夹移动过去
```

## 添加环境变量

```bash
sudo nano /etc/bash.bashrc
#写入
# Add ARM toolschain path

if [ -d /usr/local/arm-gcc/gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf ] ; then

    PATH=/usr/local/arm-gcc/gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf/bin:"${PATH}"

fi
export PATH=/usr/local/arm-gcc/gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf
export CROSS_COMPILE=arm-linux-gnueabihf-

#ctrl+x 关闭 y 确定保存
source /etc/bash.bashrc #变量生效
arm-linux-gnueabihf-gcc -v #检验是否生效

```

## 成功信息

```yaml
Using built-in specs.
COLLECT_GCC=arm-linux-gnueabihf-gcc
COLLECT_LTO_WRAPPER=/usr/local/arm-gcc/gcc-linaro-6.1.1-2016.08-x86_64_arm-linux-gnueabihf/bin/../libexec/gcc/arm-linux-gnueabihf/6.1.1/lto-wrapper
Target: arm-linux-gnueabihf
```


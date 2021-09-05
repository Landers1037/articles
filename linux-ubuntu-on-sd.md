---
title: 嵌入式设计SD卡启动linux系统
name: linux-ubuntu-on-sd
date: 2019-05-11
id: 0
tags: [linux]
categories: []
abstract: ""
---


# Xilinx-ARM-Linux-ZYNQ

项目的源代码已经上传至托管平台

[gitee](https://gitee.com/lrjgood/Xilinx-ARM-Linux-ZYNQ)

[github](https://github.com/Landers1037/arm-linux-zybo)

<!--more-->
# Xilinx-ARM-Linux-ZYNQ

项目的源代码已经上传至托管平台

[gitee](https://gitee.com/lrjgood/Xilinx-ARM-Linux-ZYNQ)

[github](https://github.com/Landers1037/arm-linux-zybo)

<!--more-->


# Xilinx-ARM-Linux-ZYNQ

项目的源代码已经上传至托管平台

[gitee](https://gitee.com/lrjgood/Xilinx-ARM-Linux-ZYNQ)

[github](https://github.com/Landers1037/arm-linux-zybo)
<!--more-->

#### 介绍

Xilinx公司的base system 硬件平台生成的，基于arm架构重新编译的linux板载操作系统

#### 软件架构

软件架构说明

- BOOT启动分区：BOOT.bin,uIamge,devicetree.dtb

- FS文件系统：Ubuntu18.04 LTS minimal rootfs

    BOOT.bin文件用于启动zybo板时加载引导程序，由板载引导加载UBOOT启动linux

    uImage文件为linux系统内核文件，用于启动系统挂载磁盘

    devicetree.dtb文件为设备树二进制文件，用于加载硬件设备

    ​	

    ```bash
    #编译设备树文件
    #利用base-system生成的dts文件，在linux下生成二进制文件
    #在设备树中修改的启动参数如下
    1:修改内核参数为CONFIG_CMDLINE="console=ttyS0,115200 mem=108M rdinit=/linuxrc root=/dev/mtdblock2"
    2：修改bootargs参数 bootargs="console=ttyPS0,115200 root=/dev/mmcblk0p2 rw earlyprintk rootfstyle=ext4 rootwait devtmpfs.mount=1";
    apt install device-tree-compiler
    dtc -I dts -O dtb -o /home/devicetree.dtb /home/system-top.dts
    ```

    

- 编译xilinix-linux内核文件：

    - 在xilinx官网获取内核[源代码](https://github.com/xilinx/linux-xlnx)

    - 安装完交叉编译环境后，使用交叉编译命令，在内核目录下，生成linux内核的映像文件

    - 交叉编译环境安装参考本文

    - 使用如下命令编译内核文件

    - ```bash
        ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- #首先配置交叉编译环境
        make xilinx_zynq_defconfig
        make
        #生成uimage
        make UIMAGE_LOADADDR=0x8000 uImage
        #无法运行上面的命令就安装编译工具 apt install u-boot-tools
        
        ```

- 文件系统

    - 因为官网下载的内核是4.*版本，所以linaro上的低版本1209已经不能使用新内核启动，所以下载linaro官网最新的文件系统版本
    - [官网](https://www.linaro.org/downloads/)
    - 在linux下使用同步命令把文件系统放入sd卡的FS区



#### 安装教程

1. 使用分区工具对sd卡进行分区，分成BOOT区（fat32格式），FS区（EXT4格式）
2. 将BOOT.bin,uImage,devicetree.dtb文件放入系统的BOOT分区
3. 使用linux系统的同步命令rysnc 把文件系统所有文件同步到FS分区内
4. `sudo rsync -av . /media/landers/fs`
5. 注意同步命令后面的为FS分区的挂载位置，使用lsblk查看或者右键sd卡属性查看

#### 使用说明

1. 把sd卡插入zybo板卡槽，将跳线帽J3放置到SD位，使用SD卡启动
2. 连接供电，使用putty或者其他串口调试工具连接zybo板，默认波特率115200
3. 按下zybo板的重启按钮，开始载入系统，在putty上观察输出

#### 成果

![](/images/linux-on-sd-1.webp))



#### 注意事项

1. 若要自己编译uboot.elf文件，需要修改zynq.h文件，去掉有关ramdisk启动的语句，因为需要sd卡启动
2. 使用xilinx官方提供的linux内核编译
3. 下载最新的xilinx官方提供的base-system系统使用vivado进行板材更新，使用sdk工具生成启动镜像文件，vivado版本低于2017.4可能出现工程不能更新的问题
4. 跳线帽一定要放在sd启动位，否则不能从sd卡启动
5. 使用linaro官方提供的ubuntu arm编译系统可能会出现内核不能挂载文件系统的问题，因为文件系统过于old
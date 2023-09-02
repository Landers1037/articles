---
title: 使用tmux
name: tmux-use
date: 2021-07-28 21:13:56
id: 0
tags: [shell,linux]
categories: [shell,linux]
abstract: ""
---

结合iTerm2使用`tmux`

![](/images/tmux-use7.png)

<!--more-->

## 安装

```bash
brew install tmux
```

![](/images/tmux-use1.png)

### 其他系统

```bash
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux
```



### 通过源码编译

```bash
git clone https://github.com/tmux/tmux.git
cd tmux && sh autogen.sh

./configure && make
```

## 会话原理

每次在iTerm中开启一个新的tab都是开启一个全新的窗口

而每次运行tmux时是打开了一个终端和tmux之间的会话，在tmux的会话上层我们可以再次输入会话命令来使上层运行的会话和终端分离

这样就不用每次创建新的窗口了，使用tmux创建的会话实际是伪窗口

使用`tmux`命令就能创建一个新的窗口

![](/images/tmux-use2.png)

底部的下标`[0]`标志着这是第0个tmux窗口

### 新的窗口

```bash
$ tmux new -s <name>
```

创建带名称的窗口

## 分离会话

使用`detach`可以分离当前窗口，退出伪窗口

```bash
$ tmux detach
```

### 重命名会话

```bash
tmux rename-session -t 0 <new-name>
```



### 查看全部窗口

```bash
$ tmux ls
```

![](/images/tmux-use3.png)

### 重连窗口

```bash
$ tmux attach -t 0
# 根据name重连 
$ tmux attach -t <name>
```

### 杀死会话

```bash
tmux kill-session -t 0
# use name
tmux kill-session -t <name>
```

### 切换会话

```bash
# 使用会话编号
$ tmux switch -t 0

# 使用会话名称
$ tmux switch -t <session-name>
```

## iTerm2配置

### 登录脚本配置

```bash
tmux ls && read session && tmux -CC attach -t ${session:-default} 
|| tmux -CC new -s ${session:-default}
```

- tmux ls: 列出已有的tmux session。
- read session: 输入你需要打开的session。
- tmux -CC attact -t ${session:-default}: 进入session。
- || tmux -CC new -s ${session:-default}: 如果session不存在则新建此session。

![](/images/tmux-use4.png)

![](/images/tmux-use5.png)

### 重载配置

```bash
$ tmux source-file ~/.tmux.conf
```

### 划分窗口

```bash
# 划分上下两格
$ tmux split-window
# 划分左右两格
$ tmux split-window -h
```

![](/images/tmux-use6.png)

### 移动光标

```bash
# 光标切换到上方窗格
$ tmux select-pane -U

# 光标切换到下方窗格
$ tmux select-pane -D

# 光标切换到左边窗格
$ tmux select-pane -L

# 光标切换到右边窗格
$ tmux select-pane -R
```

### 交换窗口位置

```bash
# 当前窗格上移
$ tmux swap-pane -U

# 当前窗格下移
$ tmux swap-pane -D
```

### 鼠标支持

鼠标支持的内容：

用鼠标点击窗格来激活该窗格
用鼠标拖动调节窗格的大小（拖动位置是窗格之间的分隔线）
用鼠标点击来切换活动窗口（点击位置是状态栏的窗口名称）
开启窗口/窗格里面的鼠标支持，用鼠标回滚显示窗口内容，按下shift的同时用鼠标选取文本，使用 ctrl+shift+c、ctrl+shift+v 的方式进行复制粘贴。
配置方式为在 `~/.tmux.conf` 文件中，增加：

```bash
set-option -g mouse on
```

### 使用github的配置

```bash
https://github.com/gpakosz/.tmux
git clone https://github.com/gpakosz/.tmux.git

$ cd
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s -f .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```

拷贝配置到`~/.tmux.conf`即可

## 快捷键

默认前缀为`ctrl + b`可以自定义配置文件

```bash
bind C-f send-prefix # 绑定 Ctrl+f 为新的指令前缀
```



### 系统指令：[#](https://www.cnblogs.com/zuoruining/p/11074367.html#2340947525)

| 前缀   | 指令   | 描述                                   |
| ------ | ------ | -------------------------------------- |
| Ctrl+b | ?      | 显示快捷键帮助文档                     |
| Ctrl+b | d      | 断开当前会话                           |
| Ctrl+b | D      | 选择要断开的会话                       |
| Ctrl+b | Ctrl+z | 挂起当前会话                           |
| Ctrl+b | r      | 强制重载当前会话                       |
| Ctrl+b | s      | 显示会话列表用于选择并切换             |
| Ctrl+b | :      | 进入命令行模式，此时可直接输入ls等命令 |
| Ctrl+b | [      | 进入复制模式，按q退出                  |
| Ctrl+b | ]      | 粘贴复制模式中复制的文本               |
| Ctrl+b | ~      | 列出提示信息缓存                       |

### 窗口（window）指令：[#](https://www.cnblogs.com/zuoruining/p/11074367.html#2918415493)

| 前缀   | 指令 | 描述                                     |
| ------ | ---- | ---------------------------------------- |
| Ctrl+b | c    | 新建窗口                                 |
| Ctrl+b | &    | 关闭当前窗口                             |
| Ctrl+b | 0~9  | 切换到指定窗口                           |
| Ctrl+b | p    | 切换到上一窗口                           |
| Ctrl+b | n    | 切换到下一窗口                           |
| Ctrl+b | w    | 打开窗口列表，用于且切换窗口             |
| Ctrl+b | ,    | 重命名当前窗口                           |
| Ctrl+b | .    | 修改当前窗口编号（适用于窗口重新排序）   |
| Ctrl+b | f    | 快速定位到窗口（输入关键字匹配窗口名称） |

### 面板（pane）指令：[#](https://www.cnblogs.com/zuoruining/p/11074367.html#908205540)

| 前缀   | 指令        | 描述                                                         |
| ------ | ----------- | ------------------------------------------------------------ |
| Ctrl+b | "           | 当前面板上下一分为二，下侧新建面板                           |
| Ctrl+b | %           | 当前面板左右一分为二，右侧新建面板                           |
| Ctrl+b | x           | 关闭当前面板（关闭前需输入y or n确认）                       |
| Ctrl+b | z           | 最大化当前面板，再重复一次按键后恢复正常（v1.8版本新增）     |
| Ctrl+b | !           | 将当前面板移动到新的窗口打开（原窗口中存在两个及以上面板有效） |
| Ctrl+b | ;           | 切换到最后一次使用的面板                                     |
| Ctrl+b | q           | 显示面板编号，在编号消失前输入对应的数字可切换到相应的面板   |
| Ctrl+b | {           | 向前置换当前面板                                             |
| Ctrl+b | }           | 向后置换当前面板                                             |
| Ctrl+b | Ctrl+o      | 顺时针旋转当前窗口中的所有面板                               |
| Ctrl+b | 方向键      | 移动光标切换面板                                             |
| Ctrl+b | o           | 选择下一面板                                                 |
| Ctrl+b | 空格键      | 在自带的面板布局中循环切换                                   |
| Ctrl+b | Alt+方向键  | 以5个单元格为单位调整当前面板边缘                            |
| Ctrl+b | Ctrl+方向键 | 以1个单元格为单位调整当前面板边缘（Mac下                     |
| Ctrl+b | t           | 显示时钟                                                     |

## 效果

![](/images/tmux-use7.png)

## 参考

[Zuorn'Blog](https://www.cnblogs.com/zuoruining/)

[github .tmux](https://github.com/gpakosz/.tmux)


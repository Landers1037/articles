---
title: rust环境配置
name: rust-start
date: 2021-08-06 11:43:16
tags: [rust]
categories: [rust]
abstract: 
---
Mac安装`rust`环境

<!--more-->

## 安装

```bash
curl https://sh.rustup.rs -sSf | sh
```

![](/images/rust-start1.png)

## 激活环境变量

```bash
source $HOME/.cargo/env
```

在没有激活前直接使用`cargo`是找不到系统路径的

![](/images/rust-start2.png)

## 新建项目

```bash
cargo new rust-new
```

![](/images/rust-start3.png)

使用Clion打开rust项目

![](/images/rust-start4.png)

首先需要安装`rust`插件

![](/images/rust-start5.png)

## 运行项目

```rust
fn main() {
    println!("Hello, world!");
}
```

![](/images/rust-start6.png)

### 使用命令方式运行

```bash
cd rust-new
cargo run
```

![](/images/rust-start7.png)


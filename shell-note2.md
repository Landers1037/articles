---
title: shell学习笔记2
name: shell-note2
date: 2020-10-30
id: 0
tags: [shell,linux]
categories: []
abstract: ""
---


Shell学习笔记2

`awk`的行列转换, `grep`高级操作

<!--more-->

## awk行列转换

使用`awk`处理数据并返回一列时

```bash
ps ax|grep python|awk '{print $1}'
447
3305
3441
3444
3445
```

我们获取了后台运行的所有`python`进程

现在我们想要把这一列的数据转为一行

使用`awk`自带的分割符

```bash
ps ax|grep python|grep -v grep|awk '{a=a$1 " "}END{print a}'
447 3305 3441 3444 3445 3495 3498 13731 
```

我们传入空格`" "`作为分隔符，最后格式化返回字符串

使用python代码解释这个逻辑

```python
pids = row.split()
line = " ".join(pids)
print(line)
```

### 举一反三

通过`,`分割输出

```bash
cat test.txt
1
2
3
4

cat test.txt | awk '{a=a$1 ","}END{print a}'
1,2,3,4,
```

### 行转为列

使用`sed`命令将对应字符替换为`\n`

```bash
cat test.txt | sed 's/" "/\n/g'
1 2 3 4
# 替换空格 为 \n
```

## grep参数

### -v

**not**选项 即排除掉符合条件的pattern

```bash
cat test.txt 
father 0
son 1

# 使用-v排除son
cat test.txt | grep -v son
father 0
```

### -e

**or**选项， 或操作

```bash
cat test.txt 
grandpa 2
father 0
son 1

# 获取son或father
cat test.txt |grep -e son -e father
father 0
son 1
```

注意：`-e`每次只能传入一个参数

### 使用`\|`也可以实现或操作

```bash
cat test.txt |grep 'son\|father'
father 0
son 1
```

注意：pattern前后的引号`''`，参数之间不能有空格

### 使用`-E`通过正则实现or

​	示例`grep -E 'pattern1|pattern2'`

```bash
cat test.txt |grep -E 'son|father'
father 0
son 1
```

### 通过`-E`实现and

示例`grep -E pattern1.*pattern2`

根据表达式获取指定的行

```bash
cat test.txt 
grandpa 2 big
father 0 big
son 1 small


cat test.txt |grep -E 'son.*small'
son 1 small

cat test.txt |grep -E 'father.*big'
father 0 big
```

可以看到第一句获取同时包含`son`和`small`的行

第二行获取同时包含`father`和`big`的行

### 尝试同时使用

**test.txt**

```bash
cat test.txt 
grandpa 2 big
father 0 big
son 1 small
```

现在获取文件中除去`son`和`0`且包含数据为`big`的行

```bash
cat test.txt | grep -v son | grep -v 0 | grep big

grandpa 2 big
```

